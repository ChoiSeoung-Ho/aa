graph TD
    subgraph Local_Node [100. 로컬 노드 : 지역 단말 / 스마트기기]
        direction TB
        A[110. 로컬 데이터 획득부<br>- 화상 이미지 및 텍스트 획득]
        B[120. 로컬 모델 학습 수행부<br>- cLDM 조건부 가상 데이터 증강<br>- 본체 동결 및 LoRA 경량 미세조정]
        C[130. 로컬 학습 결과 전송부<br>- 차분 프라이버시 노이즈 주입<br>- 초경량 LoRA 가중치만 전송]
        
        A --> B
        B --> C
    end

    subgraph Center_Node [200. 중앙 센터 : 글로벌 연합학습 서버]
        direction TB
        D[210. 글로벌 가중치 데이터 수집부<br>- 다수 노드의 암호화 가중치 병합]
        E[220. 글로벌 예측 모델 구축부<br>- 전국 범용 예측 지능 완성]
        F[230. 글로벌 설명 근거 생성부<br>- 병변 중요도 시각적 히트맵 생성<br>- VLM 기반 자연어 인과 추론 생성]
        G[240. 글로벌 결과 선별 전송부<br>- R2 성능 편차 0.3 기반 선별적 배포]
        
        D --> E
        E --> F
        F --> G
    end

    %% 노드 간 통신 흐름
    C == "보안 연합학습 통신\n(원본 유출 절대불가)" ===> D
    G -. "지능형 파인튜닝 제어\n(R2 조건 만족 시 동기화)" .-> B

    style Local_Node fill:#f0f8ff,stroke:#0055aa,stroke-width:2px,stroke-dasharray: 5 5
    style Center_Node fill:#fff0f5,stroke:#aa0000,stroke-width:2px,stroke-dasharray: 5 5
    style B fill:#e6f2ff,stroke:#333
    style C fill:#e6f2ff,stroke:#333
    style F fill:#ffe6f0,stroke:#333
