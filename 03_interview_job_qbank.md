# SK하이닉스 직무 기술 면접 — 질문 은행 (설계 · 메모리)

> 대상: Tech R&D | 설계 지원, 현 삼성 S.LSI Design Technology(RTL Sign-off·CDC/RDC/STA·검증·AI-EDA 3년+).
> 구성: 카테고리별 **질문 + [답변 키포인트] + [꼬리질문] + [브릿지]**.
> **브릿지 원칙**: 디지털설계·CDC·STA·검증 경험을 메모리 **Peri(주변회로)·HBM**이 푸는 문제에 연결. "용어 수가 아니라 *문제가 같다*는 한 문장."
> 표기: 시즌별로 바뀌는 시장 수치는 **(추정·변동)** 으로 표기. ★ = 실제 기출 빈출.

**기출 우선 포함**: 회로설계 시 고려요소 / MOSFET 동작원리 / SCE / DRAM은 왜 refresh / DRAM vs NAND / HBM 시장 SK하이닉스 위치 / 8대공정 / 신호무결성 대응 — 각 카테고리에 ★로 배치.

---

## A. 메모리 · 공정 기초 (9문항)

### A1. ★ DRAM과 NAND Flash의 근본적 차이는?
- **답변 키포인트**
  - DRAM: **휘발성**, 셀 = **1T1C**(트랜지스터 1 + 커패시터 1), **랜덤 액세스(byte/word)**, 빠른 read/write, **refresh 필요**, 시스템 **메인 메모리(working memory)**.
  - NAND: **비휘발성**, 셀에 **전하 저장**(floating gate / charge trap), **NAND string(직렬)** 구조, **페이지 단위 read/program · 블록 단위 erase**, 느린 write/erase, **P/E 마모(수명 제한)**, **스토리지**.
  - 4축 대비: 휘발성 / 셀 구조 / 액세스 단위 / 속도·내구성·용도.
- **꼬리질문**: 왜 DRAM은 커패시터를 쓰나? NAND가 3D로 간 이유는? DRAM도 3D로 갈 수 있나?
- **브릿지**: 둘 다 **셀 어레이(소자)** 바깥의 **Peri/컨트롤러 디지털 로직**(타이밍·CDC·정합성)이 존재 — 내 RTL Sign-off/검증 경험이 닿는 곳은 정확히 그쪽이다.

### A2. ★ DRAM은 왜 refresh가 필요한가?
- **답변 키포인트**
  - 1T1C 셀의 **커패시터 전하가 누설(leakage)** 로 시간이 지나면 소실 → 데이터 손실 전에 **주기적으로 읽어 다시 써줌(restore)**.
  - **retention time**(통상 64ms급) 내에 모든 row를 refresh. auto-refresh / self-refresh.
  - 미세화 → 셀 커패시턴스↓·누설↑ → refresh 부담↑. **고온일수록 누설↑** → temperature-compensated refresh.
- **꼬리질문**: self-refresh vs auto-refresh? refresh의 성능·전력 오버헤드와 완화(per-bank, fine-granularity)? RowHammer와 refresh(TRR)의 관계?
- **브릿지**: refresh 스케줄러·TRR·온도보상은 **디지털 제어 로직 + 타이밍 검증** 영역 → 내 STA/검증 직결.

### A3. DRAM 셀(1T1C) 구조와 read/write 동작은?
- **답변 키포인트**
  - WL(word line)이 access TR 게이트 제어, BL(bit line)로 입출력, 커패시터 전하가 1/0.
  - **read는 destructive**: WL on → cell 전하가 BL과 **charge sharing** → 미세 전압차를 sense amp가 증폭 → 읽으면 셀 상태가 깨지므로 **restore(write-back)** 필요.
  - BL을 Vdd/2로 **precharge** 후 sensing. cell cap : BL cap 비율이 sensing margin 좌우.
- **꼬리질문**: read가 destructive인 이유와 대응? cell capacitor를 어떻게 키우나(trench/stack, high-k)? open vs folded bitline?
- **브릿지**: WL driver·BL precharge 타이밍은 정교한 **STA·CDC 검증 대상**.

### A4. Sense Amplifier의 역할과 동작 원리는?
- **답변 키포인트**
  - read 시 cell이 BL에 만드는 **수십~수백 mV의 미세 전압차를 full logic level로 증폭**.
  - 보통 **cross-coupled latch(양의 피드백)** — precharge(Vdd/2)에서 BL/BLB의 작은 차이를 빠르게 0/Vdd로 분리.
  - **offset·sensing margin·noise**가 수율·신뢰성을 좌우, **sensing speed = access time**의 일부.
- **꼬리질문**: sense amp offset이 왜 문제이고 어떻게 줄이나(offset cancellation)? BL noise 원인? sensing margin과 cell cap의 관계?
- **브릿지**: sensing margin 확보 = **신호무결성·노이즈 마진** 문제. 내가 다룬 SI/PI·검증 마진 사고와 동형.

### A5. DRAM Peri(주변회로)란 무엇이고, 셀 어레이와 어떻게 다른가? *(브릿지 핵심)*
- **답변 키포인트**
  - 셀 어레이 = **소자·아날로그 중심**(커패시터·트랜지스터·sensing).
  - Peri = row/column **decoder**, sense amp 제어, WL/BL **driver**, command/address decode, **DLL/PLL**, **refresh controller**, **ECC**, I/O, **DFT/MBIST/redundancy** 로직 — **디지털 RTL + 아날로그 혼재**.
  - **메모리 설계에서 RTL·타이밍·CDC가 사는 곳 = Peri.**
- **꼬리질문**: Peri에서 CDC가 발생하는 대표 경계는? DLL이 메모리에서 하는 역할은? Peri 면적이 셀 효율(어레이 효율)에 주는 영향은?
- **브릿지**: **내 RTL Sign-off·CDC·STA·정합성 경험이 100% 그대로 적용되는 영역.** "메모리를 몰라서가 아니라, Peri가 곧 내가 하던 일"이라는 한 문장.

### A6. ★ 반도체 8대 공정을 설명하라.
- **답변 키포인트**
  - ① 웨이퍼 제조 ② 산화(oxidation) ③ 포토(photolithography) ④ 식각(etch) ⑤ 증착·이온주입(deposition / ion implantation) ⑥ 금속배선(metallization) ⑦ EDS(전기적 검사) ⑧ 패키징. (구분은 회사마다 약간 다름 — 통상 8단계)
  - 메모리 난도 핵심: DRAM **capacitor 형성**, NAND **고종횡비 3D 적층 식각**, 포토의 **EUV**.
- **꼬리질문**: 포토에서 EUV가 왜 중요? DRAM에서 가장 어려운 공정은? EDS와 패키지 테스트의 차이?
- **브릿지**: 내 영역은 설계(앞단)지만, **EDS/테스트 단계의 DFT·MBIST·redundancy 설계가 수율과 직결** — 설계가 제조 수율로 이어지는 연결고리를 안다.

### A7. NAND는 왜 3D(V-NAND)로 갔는가?
- **답변 키포인트**
  - 2D 평면 미세화 한계: **셀 간 간섭(coupling)**, 셀당 저장 전자 수 감소 → 신뢰성↓.
  - 3D: 셀을 **수직 적층**(현재 200단~300단대), 면적 효율↑, 셀 크기는 오히려 완화해 **신뢰성 확보**. channel hole·charge trap.
- **꼬리질문**: 단수가 올라갈수록 어려운 점(고종횡비 식각·적층 스트레스·string stacking)? QLC/PLC로 bit/cell을 늘릴 때 trade-off?
- **브릿지**: 단수↑ → page buffer·주변 제어 로직 복잡도↑ → **검증 부담↑**(내 자동화 가치↑).

### A8. ★ HBM은 왜 어려운가? (공정·패키징·검증 병목)
- **답변 키포인트**
  - 구조: DRAM die를 **8~16Hi 수직 적층 + TSV 연결 + base die(logic)**, 인터포저 위 **2.5D**로 GPU와 연결.
  - **TSV**: 수천 개 관통 비아 — 정렬·신뢰성·면적 오버헤드.
  - **적층 본딩**: SK하이닉스 **MR-MUF / Advanced MR-MUF**로 warpage·열 제어, 16Hi 고단에서 **하이브리드 본딩** 논의. 박형 die 휨 관리.
  - **열(thermal)**: 적층으로 열 밀도↑, 가운데 die 열 방출 난제 → refresh·성능 제약.
  - **수율**: known-good-die 필수 — **스택 수율 = 각 die 수율의 곱**, 한 장 불량이면 스택 전체 손실 → 고가·검증 중요.
  - **검증/테스트·SI**: 적층 후 내부 die 접근 곤란, **2048-bit(HBM4) 초광폭 I/O의 신호무결성**.
- **꼬리질문**: TSV가 와이어본딩 대비 장점? MR-MUF의 차별점? 16Hi가 8Hi보다 어려운 이유? base die를 로직 공정(TSMC)으로 가져가는 의미?
- **브릿지**: 적층·대역폭 폭증 = **SI·타이밍·테스트 복잡도 폭증**. 내 STA·CDC·DFT 검증 자동화가 정확히 이 병목을 겨냥.

### A9. DRAM 미세화의 한계와 3D DRAM 동향은?
- **답변 키포인트**
  - **capacitor scaling 한계**(충분한 capacitance 확보난), cell-to-cell 간섭, retention/refresh 악화.
  - **3D DRAM**, 4F² 셀, vertical channel transistor 등 차세대 구조 연구.
- **꼬리질문**: 왜 DRAM은 NAND처럼 쉽게 3D로 못 갔나(커패시터 적층 난이도)? 1a/1b/1c nm 노드의 의미?
- **브릿지**: 구조 전환기일수록 **새 검증 방법론·sign-off 수요**가 큼.

---

## B. 회로 · 소자 (7문항)

### B1. ★ 회로 설계 시 고려해야 할 요소는?
- **답변 키포인트**
  - **PPA**(Performance·Power·Area)를 축으로 한 trade-off.
  - timing(setup/hold), 전력(dynamic CV²f + leakage), **신호무결성(SI)·전력무결성(PI)**, crosstalk/noise, **EM(electromigration)**, 열, 신뢰성(aging), **테스트 용이성(DFT)**, **공정 변이(PVT corner)**, 면적·배선 혼잡.
  - 메모리는 특히 **power·area·yield** 민감.
- **꼬리질문**: PPA가 충돌할 때 우선순위는? PVT corner를 왜 보나? timing closure 과정은?
- **브릿지**: **DC/FC 환경 구축 + PrimeTime으로 timing closure를 직접 돌려 sign-off 감을 익힘.** 학부 AES에서 critical path MUX 제거·S-box LUT→gate 재구현으로 **면적 63%↓(27k→10k GE)** — PPA 실전.

### B2. ★ MOSFET의 동작 원리를 설명하라.
- **답변 키포인트**
  - 게이트 전압이 **gate-oxide 커패시터**를 통해 채널에 charge 유도 → **Vth 이상에서 inversion layer 형성** → source-drain 전류 제어.
  - 동작 영역: **cut-off / triode(linear) / saturation**. NMOS는 전자, PMOS는 정공.
- **꼬리질문**: triode/saturation 경계 조건(Vds vs Vgs−Vth)? body effect? Vth를 결정하는 요소?
- **브릿지**: 소자 동작 이해가 **STA의 cell delay·전력 모델**의 토대. (라이브러리 특성화 관점)

### B3. ★ Short Channel Effect(SCE)를 설명하라.
- **답변 키포인트**
  - channel length↓에서 나타나는 비이상 현상: **Vth roll-off, DIBL(Drain-Induced Barrier Lowering), punch-through, velocity saturation, hot carrier, leakage 증가**.
  - 원인: **drain 전계가 채널을 직접 제어 → 게이트 제어력 약화**.
  - 대응: halo/pocket implant, **high-k metal gate**, **FinFET / GAA**로 게이트 제어력 회복.
- **꼬리질문**: DIBL이 정확히 무엇? FinFET이 SCE를 어떻게 완화? GAA(gate-all-around)는?
- **브릿지**: SCE → **leakage↑ → DRAM retention 악화 → refresh 부담** 으로 직결 (셀 access TR에서 특히 중요).

### B4. CMOS 구조의 장점과 전력 성분은?
- **답변 키포인트**
  - PMOS pull-up + NMOS pull-down — 정상 상태에서 한쪽만 ON → **정적 전류 거의 0**, full swing, 큰 noise margin.
  - 전력 = **dynamic(CV²f) + short-circuit + leakage**. 미세화로 leakage 비중↑.
- **꼬리질문**: short-circuit current는 언제 발생? 미세화로 leakage가 왜 문제? sub-threshold slope?
- **브릿지**: leakage·dynamic 관리는 **clock gating·power gating** 저전력 기법으로 이어짐(섹션 D).

### B5. ★ 신호무결성(SI)과 전력무결성(PI), 어떻게 대응하나?
- **답변 키포인트**
  - **SI**: 신호 왜곡 없는 전달 — reflection(임피던스 부정합), **crosstalk**, ISI, ringing, jitter. 대응: **임피던스 매칭/termination**, 라우팅 간격·차폐, **차동 신호**, length matching, eye diagram 관리.
  - **PI**: 안정적 전원 — **IR drop, SSN(동시 스위칭 노이즈), ground bounce**. 대응: **PDN 설계, decoupling cap, power grid** 보강.
  - **HBM 2048-bit·10GT/s 같은 초광폭·고속 인터페이스에서 SI/PI가 핵심 병목.**
- **꼬리질문**: crosstalk를 줄이는 레이아웃 기법? termination 종류(series/parallel)? decap의 역할? HBM I/O에서 SI가 더 어려운 이유?
- **브릿지**: SI/PI = **마진 확보 = 검증 마진 사고**. 내 sign-off 방법론·검증 자동화가 고속 I/O 마진 검증에 연결.

### B6. heat sink / thermal 관리가 회로·패키지에서 왜 중요한가?
- **답변 키포인트**
  - 전력 밀도↑ → 온도↑ → **leakage↑(양의 피드백)**, **신뢰성↓(EM·aging)**, **성능↓(throttling)**. 메모리는 고온에서 **retention↓ → refresh↑**.
  - 대응: heat sink, TIM, 패키지 열경로 설계. **HBM 적층에서 가운데 die 열 방출이 난제.**
- **꼬리질문**: 온도와 leakage의 정량 관계? HBM에서 열이 refresh에 주는 영향? thermal throttling?
- **브릿지**: 열 → retention → **refresh 제어 로직 검증**으로 연결.

### B7. 트랜지스터 leakage의 종류는?
- **답변 키포인트**
  - **subthreshold, gate(tunneling), junction, GIDL(Gate-Induced Drain Leakage)**. 미세화·고온에서 증가.
  - DRAM **retention의 적**. 특히 **GIDL은 DRAM access TR**에서 중요.
- **꼬리질문**: GIDL이 DRAM에서 왜 치명적? leakage를 줄이는 소자/회로 기법은? off-state vs on-state?
- **브릿지**: leakage → retention → **refresh** (섹션 D 저전력·신뢰성과 직결).

---

## C. 디지털 · 검증 *(지원자 강점 — 깊이 있게, 메모리 연결 강조, 9문항)*

### C1. CDC(Clock Domain Crossing)와 메타스터빌리티를 설명하라.
- **답변 키포인트**
  - 비동기/위상차 클럭 도메인 간 신호 전달 시 FF **setup/hold 위반 가능 → 출력이 0/1 사이 불안정(metastable) → 시간이 지나 임의 값으로 resolve**.
  - 위험: 데이터 손상, **reconvergence 시 도메인마다 다른 값 sampling**.
  - 대응: 단일비트 **2-FF synchronizer**, 제어 **handshake**, 버스/다중비트 **async FIFO**, 포인터 **gray code**. **MTBF** 로 신뢰성 정량화.
  - 정적(structural) CDC 검증 — 동기화 구조·data stability·reconvergence 체크.
- **꼬리질문**: 왜 2-FF이고 MTBF는 어떻게 늘리나? 다중 비트는 왜 2-FF로 안 되나(skew)? gray code를 쓰는 이유? CDC false violation의 흔한 원인은?
- **브릿지**: **내 핵심 업무.** 사업부 표준 CDC sign-off 담당, **SFR-to-HW path의 CDC noise 95% 분류**, ML→LLM 기반 CDC 분류 PoC. 메모리 Peri는 **코어/I/O/DLL·PLL/refresh 등 다중 클럭 도메인** → CDC가 그대로 핵심.

### C2. RDC(Reset Domain Crossing)란 무엇이고 CDC와 어떻게 다른가?
- **답변 키포인트**
  - 비동기 **reset 도메인** 간 신호 전달 시, **reset de-assertion(해제) 타이밍**에 따른 메타스터빌리티/glitch — CDC의 reset 버전.
  - **reset tree / reset dependency** 분석이 핵심. tool 기본값이 multiple scenario를 다 보면 **false violation 폭증**.
- **꼬리질문**: CDC와 RDC의 결정적 차이? reset de-assertion이 왜 문제? false violation이 왜 폭증하나?
- **브릿지**: **내 RDC 방법론** — reset source 식별 → 의존성 시뮬 → dependency matrix → TCL constraint 자동생성의 5단계로 **RDC false violation 4.7만 → 5,795건(87.7%↓)**. 메모리 칩의 **다중 reset 도메인(power-on / soft / mode)** 에 직접 이식 가능.

### C3. ★ STA에서 setup과 hold violation의 차이는?
- **답변 키포인트**
  - **setup**: 데이터가 클럭 에지 **전에** 안정 → 위반 = 느린 경로(max delay) → **주파수를 낮추면 해결 가능**. 다음 에지 기준.
  - **hold**: 데이터가 에지 **후에도** 유지 → 위반 = 빠른 경로(min delay) → **주파수와 무관, 버퍼 삽입으로 수정**(까다로움). 같은 에지 기준.
  - **slack**, **clock skew**, OCV/PVT corner로 마진 검증.
- **꼬리질문**: hold violation을 주파수로 못 고치는 이유? clock skew가 setup/hold에 주는 영향(부호 반대)? OCV/derating?
- **브릿지**: **PrimeTime으로 timing closure 직접 수행.** **HBM 10GT/s·2048-bit I/O**는 setup/hold 마진이 극도로 빡빡 → STA sign-off가 그대로 가치.

### C4. 합성 후 정합성 검증(Logic Equivalence Check)이란?
- **답변 키포인트**
  - RTL ↔ 합성 netlist(또는 netlist↔netlist)가 **논리적으로 동일함을 formal하게 증명**(Formality / Conformal).
  - 합성·DFT 삽입·ECO·restructuring 후 **기능 보존 확인**. simulation과 달리 **exhaustive**.
- **꼬리질문**: LEC가 simulation 대비 장점? compare point와 key point? non-equivalence 디버깅 흐름? retiming/ECO 후 매칭 난점?
- **브릿지**: **내 CPU IP hardening Quick/Full EQ 이중화 환경**(Formality, **약 1,760만 line**, 인력 5→2명에도 무결성 확보). 메모리 Peri 합성·ECO 정합성 검증에 직접 이식.

### C5. UVM은 무엇이고 왜 쓰는가?
- **답변 키포인트**
  - SystemVerilog 기반 표준 검증 방법론 — 재사용 가능한 **agent/driver/monitor/sequencer/scoreboard**, **constrained-random**, **functional coverage**, layered testbench.
  - 목적: 검증 **생산성·재사용·커버리지 체계화**.
- **꼬리질문**: directed vs constrained-random? coverage-driven verification 흐름? scoreboard의 역할? sequence layering?
- **브릿지**: 메모리 컨트롤러·Peri 검증에 UVM 적용 + **내 AI 보조 검증 경험**(testbench·triage 자동화)으로 결합.

### C6. DFT(Design for Test) — scan과 ATPG를 설명하라.
- **답변 키포인트**
  - 테스트 용이성을 위해 설계 단계에서 구조 삽입: **scan chain**(FF를 shift register화), **ATPG**(자동 테스트 패턴 생성), **fault coverage**(stuck-at / transition).
  - 양산 **수율·품질의 핵심**. compression으로 테스트 시간 단축.
- **꼬리질문**: scan shift/capture 동작? stuck-at vs at-speed(transition)? scan과 CDC의 충돌(shift 시 클럭)? scan compression?
- **브릿지**: DFT = 수율 직결, 메모리는 특히 **테스트가 곧 비용** → 내 sign-off·검증 관점과 연결.

### C7. MBIST와 redundancy/repair — 메모리에서 왜 중요한가? *(가장 메모리스러운 검증)*
- **답변 키포인트**
  - **MBIST**(Memory BIST): 칩 내부 엔진이 메모리 어레이를 **March 등 패턴**으로 self-test(외부 테스터 부담↓).
  - **Redundancy/Repair**: 여분 **row/column(spare)** 으로 불량 셀 대체 → **수율 구제**. eFuse, **BIRA/BISR(self-repair)**.
  - **On-die ECC**: 미세화로 늘어난 single-bit error를 칩 내부에서 정정(DDR5).
- **꼬리질문**: March 알고리즘이 잡는 결함 유형? redundancy로 수율을 어떻게 끌어올리나? BISR 흐름? on-die ECC가 필요해진 이유?
- **브릿지**: MBIST/redundancy/ECC = **Peri 디지털 로직** → 내 DFT·검증·자동화가 직접. **브릿지 명제의 정수: "검증의 문제는 같다."**

### C8. SVA(SystemVerilog Assertion) / formal property verification은?
- **답변 키포인트**
  - assertion으로 **설계 의도(protocol, never/always)** 를 명시 → simulation·formal로 체크. immediate / concurrent.
  - **formal**: property를 **exhaustive하게 증명/반례** 도출 (시뮬레이션 미관측 영역 보강).
- **꼬리질문**: assertion이 coverage에 주는 역할? formal이 적합한 문제 유형(작은 제어 블록·protocol)? bounded vs unbounded proof?
- **브릿지**: **내 CSR spec → CDC constraint·assertion 자동생성 PoC.** 메모리 **JEDEC protocol 준수 검증**에 SVA/formal 적용.

### C9. Clock tree·skew·jitter와 메모리의 DLL/PLL 역할은?
- **답변 키포인트**
  - **CTS**로 클럭 분배, **skew**(도착 시간 차)·**jitter**가 setup/hold·CDC에 영향.
  - 메모리는 **DLL/PLL**로 내부 클럭과 외부 I/O 클럭 정렬(특히 DDR/HBM I/O의 데이터-스트로브 타이밍).
- **꼬리질문**: skew가 hold에 왜 위험한가? jitter가 고속 I/O 마진에 주는 영향? DLL과 PLL의 차이/용도?
- **브릿지**: DDR/HBM I/O 타이밍의 핵심 = STA. 내 timing closure 경험과 직접 연결.

---

## D. 저전력 · 신뢰성 (8문항)

### D1. Leakage power를 줄이는 기법은?
- **답변 키포인트**
  - **multi-Vt, power gating, body/back bias, stack effect**, 저전력 공정. (dynamic 절감은 clock gating·DVFS·voltage scaling)
  - 메모리 **standby/retention 전력**에 직결.
- **꼬리질문**: power gating의 부작용(rush current, state retention)? multi-Vt의 timing↔leakage trade-off? retention FF?
- **브릿지**: standby 전력 = **self-refresh·retention** 제어와 만남(D3).

### D2. Clock gating의 원리와 부작용은?
- **답변 키포인트**
  - 미사용 블록의 클럭 차단 → **dynamic switching power 절감**. **ICG cell**로 glitch-free 구현, enable 타이밍 중요.
- **꼬리질문**: clock gating이 STA·CDC에 주는 영향? enable 신호의 타이밍/CDC 위험? coarse vs fine-grain?
- **브릿지**: 메모리 Peri의 **bank별 clock gating**은 곧 **STA·CDC 검증 대상**(내 영역).

### D3. DRAM retention과 self-refresh 저전력 기법은?
- **답변 키포인트**
  - **retention time** 내 refresh, **self-refresh**(저전력 standby), **temperature-compensated / auto self-refresh**, **PASR(partial array self-refresh)**.
  - LPDDR은 deep power-down 등 추가 저전력 모드.
- **꼬리질문**: self-refresh에서 전력을 더 줄이는 법? 온도 보상 동작? retention 분포(weak cell)와 refresh 주기?
- **브릿지**: refresh·온도보상 = **디지털 제어 로직 검증**.

### D4. NAND의 P/E cycle 수명과 wear-leveling은?
- **답변 키포인트**
  - program/erase 반복 → **tunnel oxide 열화** → **수명(P/E cycles) 제한**. MLC<TLC<QLC로 수명·마진↓.
  - **wear-leveling**(컨트롤러), ECC, bad block 관리로 수명 연장.
- **꼬리질문**: P/E로 왜 oxide가 망가지나? TLC가 SLC보다 수명이 짧은 이유? read에도 마모가 있나?
- **브릿지**: 수명 관리 = **컨트롤러 펌웨어·로직 검증** 영역.

### D5. NAND의 Read Disturb란?
- **답변 키포인트**
  - 한 셀 read 반복 시 같은 string의 **비선택 셀 전하 상태가 미세 교란** → bit error.
  - 대응: read 횟수 모니터 → **데이터 재기록(refresh)**, **ECC**.
- **꼬리질문**: program disturb와의 차이? 3D NAND에서 disturb 특성? read reclaim?
- **브릿지**: "**반복 access가 인접 셀을 교란**"이라는 구조 → DRAM **RowHammer**와 대칭(D6). 둘 다 **모니터링·완화 디지털 로직 + 검증**.

### D6. DRAM의 RowHammer 현상과 대응은?
- **답변 키포인트**
  - 특정 row 반복 access → **인접 row 셀 전하 누설 가속 → bit flip**(신뢰성·보안 이슈).
  - 대응: **TRR(Target Row Refresh)**, 추가 refresh, on-die 완화. DDR5에서 강화.
- **꼬리질문**: 왜 인접 row가 영향받나? TRR 동작 메커니즘? DDR5의 완화 강화점? 보안 관점(공격)?
- **브릿지**: TRR = **refresh 제어 디지털 로직 + 타이밍 검증**(내 영역). read disturb(D5)와 같은 "교란-모니터-완화" 패턴.

### D7. 소자 aging(NBTI·HCI·TDDB·EM)이 설계에 주는 영향은?
- **답변 키포인트**
  - **NBTI**(PMOS Vth shift), **HCI**(hot carrier), **TDDB**(oxide breakdown), **EM**(metal 마이그레이션) — 시간·온도·전압 의존 열화.
  - 설계는 **guard band(마진)** 로 대비, EM은 power/clock net에서 특히 critical.
- **꼬리질문**: aging이 timing margin에 주는 영향(설계 시 derating)? EM이 왜 power/clock에서 중요? lifetime 예측?
- **브릿지**: aging guard band = **STA corner/마진 설정** 사고와 동일.

### D8. On-die ECC는 왜 필요해졌나?
- **답변 키포인트**
  - 미세화로 **single-bit error rate↑** → DRAM **칩 내부에 ECC 내장**(DDR5 on-die ECC)으로 신뢰성 유지. HBM은 link/내부 ECC.
- **꼬리질문**: on-die ECC vs system-level ECC의 차이? SECDED 원리? HBM의 ECC 구조?
- **브릿지**: ECC 인코더/디코더 = **Peri 디지털 로직**, 검증 대상.

---

## E. 산업 · 시사 (6문항)

### E1. 최근 HBM·AI 메모리 이슈와 HBM4는 무엇이 다른가?
- **답변 키포인트**
  - AI 가속기 수요로 **HBM 슈퍼사이클**. **HBM4**: JEDEC **2025년 4월** 표준(JESD270-4), **2048-bit I/O**(HBM3E 1024-bit의 2배), per-pin 6.4GT/s~. **base die를 로직 파운드리 공정으로** — SK하이닉스–TSMC 협업(보도상 12nm급), '메모리-파운드리 동맹'.
  - SK하이닉스 **세계 첫 HBM4 개발 완료(2025.9 발표)**, **10GT/s**(JEDEC 대비 약 25%↑), **2026년 양산**. 16-Hi 적층, **커스텀 HBM**(base die 고객 맞춤). [^1][^2][^3]
- **꼬리질문**: 2048-bit가 주는 의미? base die를 로직 공정으로 가져가는 이유(연산·고객맞춤·전력)? HBM3E와 HBM4의 핵심 차이? 커스텀 HBM이란?
- **브릿지**: 2048-bit·고속화 → **SI·타이밍·테스트 복잡도 폭증** → 내 sign-off·검증 자동화의 가치↑.

### E2. ★ HBM 시장에서 SK하이닉스의 위상은?
- **답변 키포인트**
  - **HBM 시장 선두** — 2026 기준 점유율 **약 50~60%대 (분석기관별 편차, 추정·변동)**, 엔비디아 **HBM3E(H100/H200/B100·B200) 주 공급사**.
  - **HBM4도 엔비디아 Rubin 물량 다수 확보 전망**(보도상 **약 2/3~70%**, 추정). 삼성·마이크론 추격(일부 분석은 마이크론이 삼성 추월 — 불확실).
  - 강점: **MR-MUF 적층**, **HBM3/3E/4 선제 개발(first-mover)**, 엔비디아 협업. [^4][^5][^6]
  - ※ 점유율·할당 수치는 **분기마다 변동** → 단정 금지, "선두" 정도로.
- **꼬리질문**: SK하이닉스가 HBM 선두가 된 이유는? 삼성·마이크론과의 격차 요인? 선두 유지의 리스크(수율·경쟁·고객 집중)?
- **브릿지**: 선두 유지의 본질 = **빠른 개발·검증 TAT**. 내 EDA 자동화·sign-off TAT 단축(PI/PD 20%↓ 등)이 직접 기여하는 지점.

### E3. PIM(Processing-in-Memory)이란? SK하이닉스 제품은?
- **답변 키포인트**
  - 메모리 내부에 **연산 기능** 내장 → CPU/GPU로의 **데이터 이동(메모리 월) 감소** → 대역폭·전력 효율↑.
  - SK하이닉스 **GDDR6-AiM**(Accelerator-in-Memory, 16Gbps, 1.25V — 데이터 이동 감소로 전력 절감 주장), **AiMX**(GDDR6-AiM 기반 **LLM 가속 카드**), 2025년 **AiMX/PIM2 업데이트**, "AI Memory **Creator**" 비전. [^7][^8][^9]
- **꼬리질문**: PIM이 메모리 월을 어떻게 푸나? PIM이 유리한 워크로드(memory-bound, LLM decode)? 상용화 과제(SW 스택·표준)?
- **브릿지**: PIM = **메모리 안의 디지털 연산 로직** → 설계·검증 수요. **내 디지털 설계·검증·AI 경험의 교집합.**

### E4. 경쟁사(삼성·마이크론) 대비 SK하이닉스의 차별점은?
- **답변 키포인트**
  - SK하이닉스: **HBM first-mover·점유율**, **MR-MUF 적층**, **엔비디아 락인**.
  - 삼성: 종합 반도체·**파운드리 내재화**·HBM 추격. 마이크론: 전력효율·미국 기반 공급망.
  - **객관·균형 톤. (지원자는 삼성 출신 — 비방 금지, 사실 기반)**
- **꼬리질문**: SK하이닉스가 더 잘하는 것 하나? 약점 하나? 삼성 출신으로서 본 두 회사의 문화·일하는 방식 차이?
- **브릿지**: **삼성 S.LSI(로직) + 하이닉스 메모리 = "로직·메모리 양안"** 의 균형 잡힌 시각. 양쪽 sign-off/검증 흐름을 비교해 본 경험.

### E5. 2025~2026 메모리 시황(반도체 사이클)을 어떻게 보나?
- **답변 키포인트**
  - **AI발 HBM 슈퍼사이클**로 메모리 호황, HBM은 범용 DRAM 대비 **고부가**. 범용 DRAM/NAND는 가격 변동성 존재, capex 경쟁.
  - ※ 구체 실적·가격은 **분기마다 변동 → 불확실**, 최신 IR/뉴스 확인 권고. [^10]
- **꼬리질문**: HBM 호황이 범용 메모리에 주는 영향? 공급 과잉 리스크? 다운사이클 대비 전략?
- **브릿지**: 호황기일수록 **개발 속도·검증 효율**이 경쟁력 → 내 자동화 강점의 가치.

### E6. CXL 메모리 등 차세대 인터페이스 동향은?
- **답변 키포인트**
  - **CXL**: 메모리 **풀링·확장**으로 용량·대역폭 유연성 → AI 서버 메모리 확장 대안. SK하이닉스도 CXL 메모리 개발.
  - HBM(대역폭) ↔ CXL(용량·확장)은 **보완** 관계로 자주 거론.
- **꼬리질문**: CXL이 HBM과 보완인가 경쟁인가? 메모리 풀링의 이점? CXL 컨트롤러의 검증 난점?
- **브릿지**: 새 인터페이스 = **새 컨트롤러·새 검증 방법론** 수요 → 내 방법론 개발 경험.

---

## 부록: 자기 점검 체크리스트

- [ ] **기출 8종** 30초·1분 두 버전으로 구술 가능: 회로설계 고려요소 / MOSFET / SCE / DRAM refresh / DRAM vs NAND / HBM 위상 / 8대공정 / SI 대응.
- [ ] 모든 답변 끝에 **브릿지 한 문장**("내가 하던 X = 메모리의 Y")을 붙일 수 있는가.
- [ ] **강점 3종**(CDC/RDC, STA, AI-EDA)을 메모리 Peri·HBM 좌표로 즉시 변환 가능한가.
- [ ] **갭(메모리 디바이스)** 질문에 "모릅니다"가 아니라 "**로직 관점에선 이렇게 이해했고, 메모리에선 ~로 확장될 것**"으로 답하는가.

---

## 출처 (시장·기술 사실 — 시즌별 변동 주의)

[^1]: EDN — JEDEC finalizes HBM4 standard (2025.4, JESD270-4). https://www.edn.com/jedec-finalizes-hbm4-standard/
[^2]: SK hynix Newsroom — World-First HBM4 Development Complete & Mass-Production Ready. https://news.skhynix.com/sk-hynix-completes-worlds-first-hbm4-development-and-readies-mass-production/
[^3]: Tom's Hardware — SK hynix HBM4 2,048-bit / 10 GT/s. https://www.tomshardware.com/pc-components/dram/sk-hynix-completes-development-of-hbm4-2-048-bit-interface-and-10-gt-s-speeds-promised
[^4]: Astute Group / TrendForce — SK hynix ~62% HBM share, 2026 HBM4 battle (추정·변동). https://www.astutegroup.com/news/general/sk-hynix-holds-62-of-hbm-micron-overtakes-samsung-2026-battle-pivots-to-hbm4/
[^5]: TrendForce — SK hynix ~2/3 of NVIDIA HBM4 (Rubin) (추정). https://www.trendforce.com/news/2026/01/28/news-sk-hynix-reportedly-to-supply-about-two-thirds-of-nvidia-hbm4-samsung-targets-early-delivery/
[^6]: TrendForce — SK hynix weighs TSMC logic node for HBM4 base die. https://www.trendforce.com/news/2026/03/20/news-sk-hynix-reportedly-weighs-tsmc-3nm-for-hbm4e-logic-dies-to-gain-edge-over-samsung/
[^7]: SK hynix Newsroom — GDDR6-AiM accelerator card 'AiMX'. https://news.skhynix.com/sk-hynix-debuts-first-gddr6-aim-accelerator-card-aimx-for-generative-ai/
[^8]: SK hynix Newsroom — AI Infra Summit 2025, updated AiM(AiMX3/PIM2). https://news.skhynix.com/ai-infra-summit-2025/
[^9]: SK hynix Newsroom — SK AI Summit 2025, 'AI Memory Creator' vision. https://news.skhynix.com/sk-hynix-redefines-its-vision-at-sk-ai-summit-2025-from-ai-memory-provider-to-creator/
[^10]: SK hynix Newsroom — 2026 Market Outlook: HBM-led memory supercycle. https://news.skhynix.com/2026-market-outlook-focus-on-the-hbm-led-memory-supercycle/

> ⚠️ **시장 점유율·엔비디아 할당·양산 시점**은 분석기관·시점별로 편차가 크다. 면접에서는 **"SK하이닉스가 HBM 선두"** 수준의 방향성으로 말하고, 구체 % 단정은 피할 것.
