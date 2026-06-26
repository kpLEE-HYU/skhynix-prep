# 매일 학습 캘린더 — 2주 초압축 (D-day 역산)

> 가정(본인 일정에 맞게 수정): **a!sk ≈ D+5~7**, **최종면접 ≈ D+10~14**. a!sk 먼저 → W1은 a!sk·기초, W2는 면접 deep-dive.
> 원칙: **P1(설계역량) ↔ P2(메모리) 같은 비중** + 매일 Anki 복습 + 하루 1답변 구술 녹음.
> 하루 코어 시간 = 약 60~90분(신규 15분×2 + 복습 15분 + 구술 15분 + 문서 정독 15분).

## Week 1 — a!sk + 기초 (매일 Anki 신규 + 전일 복습)
| Day | a!sk/면접 | P1 설계역량 | P2 메모리 | 산출물 정독 |
|---|---|---|---|---|
| D-14 | 자기소개·지원동기 스크립트화(02) | D_cdc ①메타·2FF | A_dram ①셀·refresh | 04 §1~3 통독 |
| D-13 | 팀워크·도전 경험(02) | D_cdc ②multi-bit·gray·FIFO | A_dram ②타이밍 tRCD/tRP/tRAS/CL | 04 §4~6 |
| D-12 | 직무 기초 12문항 구술 | E_sta setup/hold·slack | B_hbm ①TSV·스택·대역폭 | 04 §9~11 |
| D-11 | **모의 a!sk 녹화 #1** → 재생 점검 | E_sta false/MCP·ECO | B_hbm ②HBM3E/4·SK위상 | 01 팀적합 |
| D-10 | 약점 문항 재정비 | F_lowpower gating·UPF | C_nand 3D·TLC/QLC·P/E | 05 포지셔닝 |
| D-9 | 스트레스·갈등·실패 답변 | G_digital FSM·pipeline·합성 | A/B 복습(타이밍·BW 숫자) | 03 직무질문 1회독 |
| D-8 | **모의 a!sk 녹화 #2** + 환경 리허설 | H_verif UVM·coverage·DFT | J_process EUV·수율 | 04 §12 공통방어 |
| **a!sk** | 컨디션·장비 점검, 가볍게 입풀기 | I_circuit MOSFET·CMOS | K_ai_pim PIM·AI-EDA | — |

## Week 2 — 최종면접 deep-dive (구술 중심 + 약점 복습)
| Day | 면접 deep-dive | 보강 카드(약한 토픽) | 모의 |
|---|---|---|---|
| D-7 | 04 §1 RDC, §2 CDC SFR 구술(꼬리질문 자문자답) | D_cdc·E_sta 재복습 | — |
| D-6 | 04 §3 Agent, §4 Formality EQ 구술 | H_verif·G_digital | — |
| D-5 | 04 §5~8(FlowTracer·SDC·Perforce·플랫폼) 구술 | I_circuit·F_lowpower | **모의 면접 #1**(기술) |
| D-4 | 04 §9~11(AES·TFHE·SFR/IP-XACT) 구술 | A_dram·B_hbm 숫자 암기 | — |
| D-3 | 05 전체(삼성→하이닉스·신입·보안) 구술 | C_nand·J·K | **모의 면접 #2**(인성+기술) |
| D-2 | 03 직무질문 무작위 20개 구술 | 전 토픽 30초 요약만 빠르게 | — |
| D-1 | 자기소개·지원동기·핵심 수치 10개 + 1분 회사근황 | Anki 오답카드만 | 최종 점검 |
| **면접** | 두괄식·브릿지·"모르면 모른다+추론" 상기 | — | — |

## 매일 고정 루틴
1. **Anki 10~15분**: 신규 + 밀린 복습('지원서연계' 태그 100% 목표).
2. **신규 토픽 2개**(P1 1 + P2 1): `.md` 정독 → 예상Q 입으로 답.
3. **구술 1개**: 그날 핵심 질문 1개를 **녹음** → 두괄식·30~60초·수치 점검.
4. 저녁: 막힌 것 1개를 `04`/`daily_study`에서 재확인.

> 시간 부족하면 우선순위: **a!sk 스크립트 > 04 지원서 자기이해 > A_dram/D_cdc > 나머지.**
