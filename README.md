# **🏥 CareBridge**

## 📌 프로젝트 개요

기존 의료 서비스는 **응급실 정보가 제한적**이고 **단순 목록 형태로 제공**되어

상황에 맞는 선택이 어렵다는 문제가 있었습니다.

이를 해결하기 위해, 응급실 정보와 예약 기능을 통합하고

**필터 및 점수 기반 추천**을 통해 사용자가 빠르게 의사결정을 할 수 있는 서비스를 구현했습니다.

---

## 🏗️ 시스템 구조

```markdown
[공공 API]
    ↓
[Staging Table (임시 저장)]
    ↓
[데이터 유효성 검증]
    ↓
[Main Table]
    ↓
[정렬 / 필터링 로직]
    ↓
[Client 응답]
```

## 🔨주요 구현 기능

### 1️⃣ 종합 점수 기반 응급실 추천

#### 1. 기본 점수
- **거리 60% + 혼잡도 40%**
- 혼잡도 구성:
  - 응급실 일반 45%
  - 소아 응급실 20%
  - 음압 병상 20%
  - 일반 격리 10%
  - 분만실 5%
   
```python
total_score = distance_score * 0.60 + congestion_score * 0.40
```

#### 2. 뇌졸중/교통사고(거리 60% + 혼잡도 30% + 장비 10%)
```python
total_score = (
    distance_score * 0.60 +
    congestion_score * 0.30 +
    equipment_score * 0.10
)
```

#### 3. 심장/흉부(거리 80% + 혼잡도 20%)
```python
total_score = (
    distance_score * 0.80 +
    congestion_score * 0.20
)
```

#### 4. 산모/분만(거리 40% + 혼잡도 30% + 분만실 가용 여부 30%)
```python
total_score = (
    distance_score * 0.40 +
    congestion_score * 0.30 +
    birth_rate * 0.30
)
```

**✔ 설계 이유**

- 기존 응급실 서비스는 단순 거리순 또는 이름순 정렬로 인해, 병상 부족이나 장비 미보유 병원이 상위에 노출되는 문제가 발생
- 응급 상황에서는 거리뿐만 아니라 수용 가능 여부와 장비 보유 여부가 중요하다고 판단하여, 여러 요소를 반영한 점수 기반 정렬 방식 설계

**✔ 구현 내용**

- 거리, 병상 혼잡도, 장비 가용성을 각각 점수화하여 종합 점수 산출
- 응급 유형(뇌졸중, 심근경색, 산모 등)에 따라 가중치를 다르게 적용하여 상황 맞춤형 추천 구현

**✔ 기술 포인트**

- 거리 + 혼잡도 + 장비 정보를 통합한 **다중 요소 기반 점수화 로직 구현**
- 응급 유형별로 가중치를 다르게 적용한 **상황 기반 추천 시스템 설계**
- 단순 정렬이 아닌 사용자 **의사결정을 지원**하는 추천 방식 적용


### 2️⃣ 응급실 실시간 상태 조회
![응급실 메인 + 상세](https://github.com/user-attachments/assets/f246e07b-e90e-4d0e-b1ea-b9e19b06b474)

**✔ 설계 이유**

- 실시간성이 중요한 데이터와 그렇지 않은 데이터를 분리하여
  불필요한 API 호출을 줄이고 성능 최적화 및 최신 상태만 유지

**✔ 구현 내용**

- 병상 수, 장비(CT, MRI, ICU) 등 실시간 데이터 제공
- 길찾기 클릭 시 카카오 길찾기 연동

**✔ 기술 포인트**

- 스케줄러 기반 데이터 수집 자동화 (주기별 분리 처리)
- 병상 상태 데이터 : 3분 주기 자동 갱신 (실시간성 확보)
- 응급실 메세지(하루 평균 0~3건) : 30분 주기 갱신 (불필요한 호출 최소화)


### 3️⃣ 응급실 유형 / 필터 기능
![응급실 필터기능](https://github.com/user-attachments/assets/0779b5ff-6b03-4a22-a630-77d735b16c21)

<img width="728" height="588" alt="image" src="https://github.com/user-attachments/assets/2d4dfef0-e302-4955-9780-a3d54378b1a2" />


**✔ 설계 이유**

- 응급 상황에서 빠른 의사결정을 돕기 위해 복잡한 정보 탐색 과정을 최소화

**✔ 구현 내용**

- 복잡한 의료 판단 없이도 **상황에 맞는 응급실을 즉시 선택**할 수 있도록 설계
- 응급 상황에 필요한 **장비 기준으로 응급실을 빠르게 선별**할 수 있는 필터 기능 구현
- CT, MRI, ICU 등 주요 응급 장비의 **실시간 가용 여부**를 반영하여 결과 제공

**✔ 기술 포인트**

- AJAX 기반 비동기 요청으로 페이지 리로드 없이 데이터 갱신
- 조건 기반 필터링 로직 구현

---
## **⚠️ Trouble Shooting**

### 1️⃣ API 데이터 누락 문제

**🔴 문제**

공공 API 데이터에서 병상 수, 장비 정보가 누락되거나 형식이 일관되지 않는 문제가 발생

**🔴 해결(코드)**

- Raw 데이터를 그대로 사용하지 않고 **Staging 테이블에 먼저 적재**
- 이후 유효성 검증 및 변환 후 **Main 테이블에 반영**

<img width="333" height="223" alt="image" src="https://github.com/user-attachments/assets/5d5f086b-93cf-47de-9ef0-bdacb83a1b0e" />

**💻** `fetch_emergency.py` (325 ~ 479줄)

```python
def merge_staging_to_main():
    # 중복 제거 (병원 + 시간 기준)
    unique_map = {}
    for st in staging_list:
        key = (st.hospital.er_id, st.hvdate)
        if key not in unique_map:
            unique_map[key] = st

    # 최신 데이터만 유지
    ErStatus.objects.all().delete()
```

**🔴 결과**

- 잘못된 데이터 필터링
- 최신 데이터만 유지
- 서비스 신뢰성 확보

### 2️⃣ 외부 API 장애 대응

**🔴 문제**

API 요청 실패 시 서버 프로세스가 중단될 가능성 존재

**🔴 해결(코드)**

- **timeout 설정**으로 무한 대기 방지
- 예외 발생 시 **빈 데이터 반환**

<img width="700" height="367" alt="image" src="https://github.com/user-attachments/assets/c9791027-b5e7-4007-b93d-c54eb3204ab8" />

**🔴 결과**

API 장애 상황에서도 서비스 안정성 유지

### 3️⃣ 혼잡도 데이터 불일치 문제

**🔴 문제**

메인 페이지와 상세 모달에서 같은 병원의 혼잡도 값이 다르게 표시됨

**🔴 해결(코드)**

혼잡도 계산을 **서버에서 단일 처리** ➡ 프론트는 결과만 렌더링하도록 변경

<img width="441" height="329" alt="image" src="https://github.com/user-attachments/assets/f911ac89-cec9-48d4-a4ac-048b0e1e0a11" />

<img width="441" height="329" alt="image" src="https://github.com/user-attachments/assets/7d0030f9-bee8-4357-9369-3f0c91efdbe6" />

**🔴 결과**

데이터 일관성 확보

---

## 발표영상 - 12분 30초 ~ 17분 15초(개인발표)

[서비스 시연 영상(바로가기)](https://youtu.be/T2mVEn3PbGM?si=QJed2NFk4qqhC6LP)

---

## **DB 구조 (ERD)**

<img width="1589" height="1092" alt="image" src="https://github.com/user-attachments/assets/ea278956-a180-4e37-9bb1-3ea8c633be9f" />
