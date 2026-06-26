# 01. 팀 적합도 & 브릿지 전략 — 이관표

> 한 문장 결론: **"Digital 설계검증 + Sign-off Methodology + AI 자동화"를 메인 정체성으로 세우고, 그 경험이 곧 메모리 Peri/HBM Base Die 디지털 설계가 푸는 문제임을 한 문장으로 연결한다.**
> 근거 명제(`ref/직무핏.pdf`): *"메모리 용어의 수가 아니라, 내 경험의 문제가 메모리가 푸는 문제와 같다는 한 문장이 직무핏을 만든다."*

---

## 1. SK하이닉스 "설계(Design)"의 구조 — 12개 세부 롤
신입(Talent hy-way)은 **통합 '설계'로 지원 → 입사 후 배치**. 따라서 "어느 롤 언어로 나를 말하느냐"가 핵심.

`jd/JD_Tech R_D.pdf`에 명시된 설계 세부 롤:
① 회로설계(HBM) ② HBM Digital Design(Front-end) ③ (Back-end) ④ **(RTL Design)** ⑤ **(SoC Design)** ⑥ **(검증)** ⑦ HBM Logic Die/DFM ⑧ 회로설계(DRAM) ⑨ Interface 설계 ⑩ Analog Design ⑪ 배치설계(+설계자동화) ⑫ 설계검증

이 중 **디지털·검증·자동화/방법론 계열(④⑤⑥⑪⑫)**이 내 경력과 직접 맞물린다.

---

## 2. 적합도 매핑 (JD 원문 ↔ 내 경력)

| 순위 | 타깃 롤 | JD 원문 근거 | 내 경력 대응 | 신입 적격 |
|---|---|---|---|---|
| **1A (현실 착지)** | **설계검증 / HBM Digital(검증)** | "세계 최고 수준 **검증 방법론과 자동화 환경**… SV·UVM 재사용 검증환경… **검증 수렴(Sign-off)**… **검증 자동화 스크립트·새 방법론 도입**" / "Custom IP에 대한 **UVM Agent 설계/구현**" | RTL Sign-off 방법론, 검증 자동화, AI 검증 | ✅ #Junior&Expert |
| **1B (경력 정합)** | **배치설계·설계자동화 / Design Methodology / CAD** | "설계 **자동화 기반 차세대 개발환경**… 다양한 **Sign-off 체계 구축·검증**" · "**Digital Transformation 통한 설계 자동화·AI 적용**… **설계DB Sign-off 실행**… 자동화 기반 회로설계·검증 업무효율화" · "**Design Methodology 개발(Digital Implementation Flow)**… Layout 자동화 스크립트(TCL/Python)" · "**Logic Block의 CDC/RDC 검증·IP 개선**, 합성/STA" · "**AI 활용 Implementation flow 구축·Methodology 개발/개선**" | 내 현직 업무와 **거의 1:1** (Sign-off·CDC/RDC·자동화·AI) | 일부 #Expert (관심 신호로 활용) |
| 2 | HBM Digital Design(RTL/BE/SoC) | "HBM Base Die RTL 설계·검증, Logic/Macro Floorplan, **합성·STA·DFT**, DRAM Memory Controller, PDN(EM/IR) signoff" | STA·EQ·합성 정합성·DFT 이해 | 주로 #Expert |
| 2 | 회로설계(DRAM/HBM) | "내/외부 요구사양 부합 **회로 설계**, 양산 설계품질·수율, **Full Custom + Digital Co-Design**" | 학부 암호HW 회로설계·PPA 최적화 | ✅ #Junior&Expert |
| 참고(2지망) | Product Engineering | 데이터·자동화·분석 (직무핏 가이드: "PE엔 데이터 분석을 앞에 세워라") | 측정/분석 자동화, 통계 | — |

**추천 = dual-track**: 면접 도입은 **1A(설계검증)**로 "신입으로서 바로 기여 가능한 자리"를 잡고, 꼬리질문이 깊어지면 **1B(자동화·방법론)**로 "이미 이 일을 해봤다"를 드러낸다. 회피: **CIS(사업 철수)**, 순수 **Analog/배치 full-custom**(SPICE 진입장벽).

> ⚠️ 주의: 1B 롤 상당수는 #Expert(경력) 태그. 신입 전형에선 "그 일을 신입으로 시작해도 좋고, 방법론 경험이 설계 이해를 돕는다"는 톤으로. 경력 과시가 아니라 *설계를 더 잘 이해하는 신입*으로 포지셔닝.

---

## 3. 브릿지 narrative 12선 (내 프로젝트 → 메모리가 푸는 문제)
면접에서 각 경험을 말한 뒤 **마지막 한 문장**으로 메모리에 착지시킨다.

1. **RDC 부분집합 false-path 알고리즘(-87.7%)** → 메모리 SoC/HBM Base Die도 HW/SW reset이 얽혀 RDC noise가 폭증한다. *"같은 reset 의존성 매트릭스 방법을 메모리 Peri/컨트롤러에 그대로 적용해 sign-off 신뢰도를 올릴 수 있습니다."*
2. **CDC SFR-to-HW(CSR spec 표준화, noise 95% 분류)** → HBM Base Die·메모리 컨트롤러의 configuration register가 clock domain을 건너는 동일 문제. *"메모리 설계의 SFR→logic CDC도 같은 spec-driven 방식으로 푼다."*
3. **RTL Sign-off Agent(LLM+RAG+deterministic 분기, false −20%)** → 메모리 설계 sign-off(Lint/CDC/STA)의 false violation도 동일 구조. *"AI를 사람 판단 위가 아니라 아래에 두는 보조도구로."*
4. **Formality Full/Quick EQ(1,760만 line, 인력 5→2)** → 메모리 IP hardening·restructuring 후 기능 정합성 보장에 그대로 필요.
5. **Backend 자동화(FlowTracer, PI/PD TAT −20%)** → 메모리 PD/PI signoff의 긴 runtime·hand-off idle 단축.
6. **SDC signoff 평가(Gencellicon/Xcelium TCV)** → 메모리 timing constraint(MCP/false path) 정합성 검증 = silicon 전 bug 차단.
7. **SoC 통합환경(Perforce, 3개 해외 site, Human Error −50%)** → 메모리 다거점 R&D의 설계 release·버전 무결성.
8. **AES/ECC PPA 최적화(critical path MUX 제거, S-box LUT→logic, 27k→10k GE)** → 메모리 Peri 회로의 면적·타이밍 최적화 감각.
9. **TFHE Key Switching IP 1-cycle 단축(1저자)** → datapath/연산 가속 설계 감각 → PIM/GDDR6-AiM·HBM 연산과 연결.
10. **학부 5-stage MIPS·SOC설계·VLSI** → "방법론만"이 아니라 RTL 설계 기본기 보유 증명.
11. **AI Skill 플랫폼 500명+·조직혁신상** → SK하이닉스 메모리 R&D의 AI Transformation/CAD 확산에 즉시 기여.
12. **주도적 SRAM 설계·DC/FC/PrimeTime timing closure 직접** → 설계자 시각 보유 → 검증/방법론을 설계 품질로 연결.

---

## 4. 핵심 갭 & 보완책
- **갭 1: 메모리 디바이스 지식** (DRAM cell/refresh/타이밍, HBM stack/BW, NAND) → `daily_study/A_dram·B_hbm·C_nand` 매일 암기로 2주 내 보강.
- **갭 2: "방법론만 했지 설계 가능?"** → 자소서1 근거(주도적 스터디로 SRAM 설계·검증, DC/FC/PrimeTime 직접) + "설계를 이해해야 좋은 방법론이 나온다"는 역논리. `04`에서 답변 스크립트화.
- **갭 3: 로직(S.LSI) ≠ 메모리** → 차이를 먼저 인정하고(아래) 공통점(디지털 IP·CDC·STA·DFT·검증)으로 잇기.
  - 로직: 다양한 기능 IP·고성능·foundry 다변. 메모리: 셀/Peri 중심·초미세·수율·리프레시·HBM은 Base Die에 로직 집약.
  - 공통: **합성·STA·CDC/RDC·DFT·검증 무결성** = 내가 매일 다룬 것.

---

## 5. 2지망(Product Engineering) 한 줄 정리
측정·수율·신뢰성 데이터 분석 직무. 내 *자동화·데이터·hazard 조기탐지* 경험을 "데이터로 양산 품질을 끌어올린다"로 재포장 가능. 단, 면접 메인은 설계로 유지.

---
*다음 파일: `04_resume_selfmastery.md`(지원서 완전 이해), `03_interview_job_qbank.md`(직무질문), `daily_study/`(매일 암기).*
