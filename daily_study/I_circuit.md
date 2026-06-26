# Circuit & Device — 회로/소자
> 📌 MOSFET 동작영역·CMOS·leakage·sense amp·차동증폭·PLL/DLL·신호무결성 등 디지털 위 아래의 회로/소자 기본기 · 🏷️ P1 · 🧠 메모리 브릿지: DRAM read의 핵심은 sense amp(미소 charge 감지·증폭), DDR/HBM 클럭·DQS 정렬은 DLL, 고속 인터페이스 품질은 SI/PI(crosstalk·IR drop). 지원자의 메모리 갭을 메우는 토픽.

## 핵심 개념
- **MOSFET 3영역**(NMOS 기준, V_ov=V_GS−V_th):
  - **cutoff**: V_GS<V_th → 채널 없음, I_D≈0(실제론 subthreshold leakage, exp 의존).
  - **triode/linear**: V_GS>V_th, V_DS<V_ov → **I_D=μC_ox(W/L)[(V_GS−V_th)V_DS−V_DS²/2]**, 저항처럼.
  - **saturation**: V_GS>V_th, V_DS≥V_ov → **I_D=½μC_ox(W/L)(V_GS−V_th)²(1+λV_DS)**, pinch-off, ~정전류.
- **CMOS 인버터**: PMOS pull-up+NMOS pull-down. 정상상태에 한쪽이 항상 OFF → **VDD→GND DC 경로 없음 → 정적전류≈0**. 전력은 switching(+leakage)에서만.
- **Leakage**: subthreshold(V_GS<V_th, weak inversion, exp, ~60mV/dec)·gate(얇은 oxide tunneling)·junction/GIDL. 온도·미세공정에서 급증.
- **Short Channel Effect**: L 축소 시 V_th roll-off, **DIBL**(drain이 barrier 낮춤), velocity saturation, hot carrier, leakage 증가.
- **Sense Amplifier**: DRAM cell의 미소 charge가 bitline에 charge sharing으로 작은 ΔV(수십 mV) 생성 → cross-coupled inverter latch가 full rail로 증폭. read는 **destructive → write-back(restore)** 동반.
- **차동증폭기**: 두 입력의 **차**를 증폭, common-mode 제거(CMRR↑). 노이즈 내성↑. sense amp·op-amp 입력단 기반.
- **PLL**: phase detector+charge pump+loop filter+**VCO**+divider. 주파수 합성/체배 가능, VCO라 jitter 누적.
- **DLL**: phase detector+**voltage-controlled delay line**. 위상만 정렬(체배 없음, 출력f=입력f), oscillator 없어 jitter 누적 적고 안정. **DDR의 DQS/clock 정렬**에 사용.
- **Signal/Power Integrity**: crosstalk(인접 net 결합→noise·delay), IR drop(전원망 저항 전압강하→느려짐), EM(electromigration), PI=전원 안정(decap/grid), SI=신호 청결.

## 예상 면접 Q&A

### Q1. MOSFET의 세 동작 영역과 전류식을 설명하세요.
**A.** NMOS 기준 overdrive V_ov=V_GS−V_th로 봅니다. (1)cutoff: V_GS<V_th로 채널이 없어 I_D≈0(실제론 지수적 subthreshold leakage). (2)triode/linear: V_GS>V_th이고 V_DS<V_ov로 채널이 끝까지 형성, I_D=μC_ox(W/L)[(V_GS−V_th)V_DS−V_DS²/2]이며 V_DS에 거의 비례해 저항처럼 동작. (3)saturation: V_DS≥V_ov로 drain 쪽 채널이 pinch-off, I_D=½μC_ox(W/L)(V_GS−V_th)²(1+λV_DS)로 V_DS에 거의 무관한 정전류(λ는 채널 길이 변조). 디지털 스위치는 ON 시 주로 triode, 아날로그 증폭은 saturation을 씁니다.
- ↳ 꼬리질문: 디지털 게이트에서 MOS는 주로 어느 영역? → 스위칭 도중 saturation을 거쳐 ON 정착 시 triode(낮은 R_on), OFF는 cutoff.
- 🧠 메모리 브릿지: sense amp·아날로그 peri는 saturation 바이어스, 디지털 로직은 triode/cutoff 스위칭.

### Q2. CMOS 인버터의 정적 전류가 거의 0인 이유는?
**A.** CMOS 인버터는 위에 PMOS, 아래 NMOS가 직렬인데 입력이 0이든 1이든 정상상태에서는 둘 중 하나가 반드시 OFF입니다(입력 0→NMOS off, 입력 1→PMOS off). 따라서 VDD에서 GND로 흐르는 DC 경로가 없어 정적 전류가 거의 0이고, 전력은 입출력이 toggle하며 캐패시터를 충방전하는 dynamic power와 누설(leakage)에서만 발생합니다. 이 "정적 0 전류"가 CMOS가 표준이 된 이유입니다.
- ↳ 꼬리질문: 그럼 전이 순간에는? → 입력이 V_th~VDD−|V_tp| 사이일 때 둘 다 잠깐 ON → short-circuit current가 흐름(빠른 transition으로 최소화).
- 🧠 메모리 브릿지: 메모리 peri 디지털도 CMOS라 정적전력은 leakage가 좌우 → 저전력 설계와 연결.

### Q3. 누설(leakage)의 종류와 short channel effect를 설명하세요.
**A.** 주요 leakage는 (1)subthreshold: V_GS<V_th여도 weak inversion으로 지수적으로 흐르는 전류(이상적 기울기 ~60mV/dec, 온도에 민감), (2)gate leakage: oxide가 얇아져 전자가 tunneling, (3)junction/GIDL입니다. short channel effect는 채널 길이 L이 짧아질 때 생기는 현상으로 V_th가 낮아지는 roll-off, drain 전압이 barrier를 낮추는 DIBL, velocity saturation, hot carrier 열화가 있고, 모두 leakage와 변동을 키웁니다.
- ↳ 꼬리질문: leakage가 가장 큰 영향을 받는 변수? → 온도(지수적)와 Vt. 그래서 multi-Vt·power gating이 대책.
- 🧠 메모리 브릿지: DRAM cell leakage가 곧 보존시간 저하 → refresh 주기를 결정.

### Q4. DRAM의 sense amplifier 원리를 설명하세요.
**A.** DRAM cell은 1T1C로 capacitor에 적은 charge만 저장합니다. 워드라인이 켜지면 cell capacitor가 미리 precharge된(보통 VDD/2) bitline과 charge sharing해 bitline에 아주 작은 전압차(수십 mV)만 만듭니다. 이 미소 차이를 reference bitline과 비교해 **cross-coupled inverter latch(sense amp)**가 양의 피드백으로 full rail(0/VDD)까지 증폭해 데이터를 판별합니다. 이때 cell의 charge가 소모되므로 **read는 destructive**이고, 증폭된 값을 다시 cell에 써 주는 write-back(restore)이 같은 동작에 포함됩니다.
- ↳ 꼬리질문: bitline을 VDD/2로 precharge하는 이유? → 0/1 양방향 미소 신호를 대칭으로 감지하고 전력·속도를 최적화하기 위해(open/folded bitline 구조).
- 🧠 메모리 브릿지: sense amp는 DRAM 성능·수율의 심장. "DRAM read=미소 charge 감지+증폭+복원"이 핵심 한 줄.

### Q5. 차동증폭기를 쓰는 이유는?
**A.** 차동증폭기는 두 입력의 차이만 증폭하고 두 입력에 공통으로 실린 신호(common mode: 전원 노이즈·온도 drift 등)는 억제합니다(높은 CMRR). 덕분에 노이즈 내성이 좋고 기준 대비 미소 신호를 안정적으로 키울 수 있어, sense amp·op-amp 입력단·고속 IO 수신기에 두루 쓰입니다. 단일단보다 PSRR·linearity도 유리합니다.
- ↳ 꼬리질문: CMRR이 무엇? → (차동이득)/(공통모드이득). 클수록 공통 노이즈를 잘 제거.
- 🧠 메모리 브릿지: 고속 메모리 IO(차동 strobe·수신기)와 sense amp가 모두 차동 구조 → 노이즈 환경에서 미소 신호를 살림.

### Q6. PLL과 DLL의 차이, 그리고 DDR에서 왜 DLL을 쓰나요?
**A.** PLL은 phase detector·charge pump·loop filter·**VCO**·divider로 구성돼 입력 대비 주파수를 체배/합성할 수 있지만, VCO가 발진기라 jitter가 누적됩니다. DLL은 VCO 대신 **전압제어 지연선(delay line)**으로 위상만 맞추므로 주파수 체배는 못 하지만(출력f=입력f) 발진기가 없어 jitter 누적이 적고 위상 정렬이 안정적입니다. DDR SDRAM은 출력 DQ/DQS를 외부 clock에 정확히 정렬해 skew를 줄이려고 on-die DLL을 써서 data eye를 확보합니다.
- ↳ 꼬리질문: 주파수를 바꿔야 하면? → PLL(체배). 위상만 맞추면 DLL. 둘을 조합하기도.
- 🧠 메모리 브릿지: DQS(data strobe) 정렬·de-skew가 DDR/HBM 고속 전송의 타이밍 마진을 만든다 → STA의 data eye와 연결.

### Q7. signal integrity와 power integrity(crosstalk, IR drop)를 설명하세요.
**A.** signal integrity는 신호가 왜곡 없이 전달되는지에 관한 것으로, 대표 문제가 **crosstalk**(인접 net 간 용량·유도 결합으로 victim에 noise glitch가 유기되거나, aggressor 동시 천이로 실효 결합용량이 바뀌어 지연이 빨라지거나 느려짐)입니다. power integrity는 전원이 안정적인지로, **IR drop**(전원/접지망 저항에 전류가 흘러 국소 전압이 떨어져 셀이 느려지거나 오동작)과 di/dt 노이즈, electromigration이 있습니다. 대책은 spacing·shielding, decap 추가, 전원 grid 강화이고, STA는 SI delay/noise를 sign-off에 포함합니다.
- ↳ 꼬리질문: crosstalk가 timing에 주는 영향? → 결합 net이 반대로 천이하면 victim 지연 증가(worst), 같이 천이하면 감소 → PrimeTime SI가 이를 반영.
- 🧠 메모리 브릿지: HBM은 TSV·interposer로 초고밀도라 SI/PI(crosstalk·IR drop·전원 노이즈) 관리가 동작 속도를 좌우.

## 30초 암기 요약
- MOSFET: cutoff(V_GS<V_th, I≈0)·triode(V_DS<V_ov, 저항, I=μC_ox(W/L)[(V_ov)V_DS−V_DS²/2])·saturation(V_DS≥V_ov, 정전류, I=½μC_ox(W/L)V_ov²(1+λV_DS)).
- CMOS 인버터: 한쪽 항상 OFF → DC 경로 없음 → 정적전류≈0. 전력=switching+leakage(+전이 시 short-circuit).
- leakage=subthreshold(exp, ~60mV/dec)+gate tunneling+junction. SCE=Vt roll-off·DIBL·velocity sat·hot carrier.
- DRAM sense amp=미소 ΔV(수십 mV)를 cross-coupled latch로 full rail 증폭, read는 destructive→write-back.
- 차동증폭=차만 증폭·common mode 억제(CMRR↑), 노이즈 내성. sense amp/IO 수신기 기반.
- PLL=VCO로 주파수 체배(jitter 누적), DLL=delay line으로 위상 정렬(체배 X, jitter 적음). DDR DQS 정렬=DLL.
- SI=crosstalk(noise·delay), PI=IR drop·di/dt·EM. HBM/DDR 고속일수록 치명.
