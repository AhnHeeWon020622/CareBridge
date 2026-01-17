# **🏥 CareBridge**

## 공공의료 데이터 기반 병원·응급실·예약 통합 관리 시스템

---

## 주요 구현 기능

### 1. 응급실 실시간 상태 조회

- 병상 수, 가용 장비 정보(CT, MRI, ICU 등) 실시간 제공
- 데이터 성격에 따라 수집 주기 분리
    - 병상 상태: 약 3분 주기
    - 병원 공지/메시지(하루 평균 3건 이하 이벤트성 데이터): 약 30분 주기
- 최신 상태만 유지하도록 설계

![응급실 메인 + 상세](https://github.com/user-attachments/assets/f246e07b-e90e-4d0e-b1ea-b9e19b06b474)


### 2. 응급 유형 / 장비 필터 기능

![응급실 필터기능](https://github.com/user-attachments/assets/0779b5ff-6b03-4a22-a630-77d735b16c21)


- 복잡한 의료 판단 없이도 **상황에 맞는 응급실을 즉시 선택**할 수 있도록 설계
- 응급 상황에 필요한 **장비 기준으로 응급실을 빠르게 선별**할 수 있는 필터 기능 구현
- CT, MRI, ICU 등 주요 응급 장비의 **실시간 가용 여부**를 반영하여 결과 제공

---

## 발표영상

[서비스 시연 영상(바로가기)](https://youtu.be/T2mVEn3PbGM?si=QJed2NFk4qqhC6LP)

---

## **DB 구조 (ERD)**

<img width="1589" height="1092" alt="image" src="https://github.com/user-attachments/assets/ea278956-a180-4e37-9bb1-3ea8c633be9f" />
