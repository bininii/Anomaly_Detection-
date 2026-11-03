# 🏭 산업 현장의 조명 변화에 강건한 이상탐지 모델 개선
> **Stable Diffusion 3.5 기반 데이터 합성 및 가성 불량(False Positive) 저감 프로젝트**

---

## 📌 1. 프로젝트 개요 (Overview)
- **프로젝트 주제:** 제조 현장의 조명 변화(세기, 색온도 등)로 인해 발생하는 **공변량 변화(Covariate Shift)** 문제를 해결하고, 정상 제품을 불량으로 오인하는 **가성 불량률(False Positive Rate, FPR)**을 낮추기 위한 데이터 증강 및 모델 검증 프로젝트
- **핵심 목표:** 
  - 하드웨어 독립성과 실시간 추론 제약(밀리초 단위)을 만족하는 경량 이상탐지 모델 **EfficientAD** 선정 및 적용
  - **Stable Diffusion 3.5 (img2img)**를 활용한 다양한 조명 환경의 정상 데이터 합성 및 품질 검증
  - 결함 탐지 민감도(Recall $\ge$ 95%)를 유지한 상태에서 가성 불량률(FPR) 개선 효과 정량 검증
- **참여 역할:** 데이터 증강 파이프라인 구축, FID 기반 데이터 밸리데이션(검증) 로직 설계, 성능 지표 분석 및 한계점 고찰

---

## 🔄 2. 전체 파이프라인 및 아키텍처 (Pipeline)

```mermaid
graph TD
    A[원본 데이터셋 수집] -->|Covariate Shift 문제 정의| B(Stable Diffusion 3.5 기반 조명 데이터 합성)
    B -->|img2img 및 파라미터 스윕| C{FID 기반 데이터 밸리데이션}
    C -->|형태 붕괴 Bad Case 필터링| D[최적 합성 데이터셋 구축]
    D -->|비율별 치환 학습 25/50/75%| E(EfficientAD 모델 학습)
    E -->|Recall 95% 고정 및 FPR 검증| F[최종 성능 평가 및 가성 불량률 저감 확인]

---



## 🛠️ 3. 데이터 증강 및 품질 검증 (Data Generation & Validation)

### ① 조명 조건별 데이터 생성 결과
* **Stable Diffusion 3.5 img2img 스윕**을 통해 앰플(Vial) 및 캔 제품에 다양한 조명(밝음, 어두움, 따뜻한 색조 등)을 자연스럽게 합성했습니다.
<img width="1242" height="638" alt="vial_can_result" src="https://github.com/user-attachments/assets/64e8eb72-f529-41c4-bbb6-98581f530405" />





### ② FID 기반 Bad Case(형태 붕괴) 필터링
* 생성형 AI가 과도하게 작동해 형태가 일그러지거나 글자가 깨지는 이미지(Bad Case)를 걸러내기 위해 **FID(Frechet Inception Distance)** 임계값을 설정하고 스크리닝을 거쳤습니다.

<img width="1051" height="531" alt="스크린샷 2026-11-03 161838" src="https://github.com/user-attachments/assets/662109ee-1183-42b2-b07f-b557f1c18db2" />
<img width="1205" height="663" alt="스크린샷 2026-11-03 161847" src="https://github.com/user-attachments/assets/22952068-02e1-458c-b2ed-2cb093cfe774" />


---

## 📈 4. 주요 실험 결과 (Results)
* **vial 품목 실험 결과 예시** (데이터 치환 비율별 성능 변화):
  * **Baseline (0% 합성 데이터):** AUROC 0.7777 / Recall 0.9524 / FPR 0.8857
  * **합성 데이터 75% 치환 시:** AUROC 0.8122 / Recall 0.9524 / FPR **0.7702 (-11.5% 개선)**

<img width="1168" height="665" alt="스크린샷 2026-11-03 162000" src="https://github.com/user-attachments/assets/1ce48543-14c2-4fc0-8aca-21237bd44b6a" />
<img width="1244" height="634" alt="스크린샷 2026-11-03 162009" src="https://github.com/user-attachments/assets/b834444c-f4f1-451e-973a-c782b3ec7433" />
<img width="1273" height="631" alt="스크린샷 2026-11-03 162017" src="https://github.com/user-attachments/assets/6e5dc644-6f17-46fd-8a13-95247d298e83" />



---

## ⚠️ 5. 프로젝트 한계점 및 시사점 (Limitations & Critical Review)
* **품목별 편차 분석:** 일부 품목(Can, Rice 등)에서는 베이스라인 자체의 성능 한계로 인해 데이터 증강 효과를 충분히 이끌어내지 못한 한계 확인
* **통계 지표의 맹점 인지:** FID 같은 통계적 거리 지표가 이미지의 육안상 품질이나 실제 현장 유효성을 100% 보장하지 않는다는 점을 분석하고, 향후 실제 현장 데이터셋을 통한 교차 검증의 필요성
