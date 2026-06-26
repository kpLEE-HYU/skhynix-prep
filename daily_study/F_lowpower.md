# Low Power Design — 저전력 설계
> 📌 dynamic·static power를 clock/power gating, multi-Vt, DVFS, power domain(UPF) 등으로 줄이는 설계 기법 · 🏷️ P1 · 🧠 메모리 브릿지: LPDDR/모바일 DRAM의 self-refresh·power-down·deep-sleep, retention, 전압 island가 곧 저전력 설계. HBM Base Die 디지털도 clock gating·power domain으로 전력 관리.

## 핵심 개념
- **Dynamic power = α·C·V²·f** (switching) + short-circuit power. α=activity factor. V가 제곱이라 전압 낮추기가 제일 효과적.
- **Static power = leakage** (V·I_leak). subthreshold(지배적)·gate·junction. 온도·미세공정에서 급증.
- **Clock gating**: idle FF에 clock 차단 → clock tree+FF switching 절감(최대 dynamic 절감 기법). **ICG 셀**(latch-based)로 enable glitch 방지. RTL/합성 자동 삽입.
- **Power gating**: idle 블록 전원 차단(header=PMOS→VDD / footer=NMOS→GND sleep TR, MTCMOS) → **leakage 제거**. isolation+retention+전원 시퀀싱 필요.
- **Multi-Vt**: high-Vt(느림·저leakage)는 non-critical에, low-Vt(빠름·고leakage)는 critical path에 → speed/leakage trade-off. 합성/STA가 VT swap.
- **DVFS**: 성능 불필요 시 V·f 동시 하향. P∝V²f라 절감 큼. PMU+adaptive.
- **Voltage island/power domain**: 영역별 다른/스위치 가능한 전압.
- **UPF(IEEE 1801)**: power intent(domain·supply·isolation·retention·level shifter)를 RTL과 분리 기술.
- **Retention flop**: always-on supply의 balloon/shadow latch에 state 보관 → power-down 후 복원.
- **Isolation cell**: power-down 도메인 출력을 0/1로 clamp → X/float가 켜진 도메인으로 못 들어가게. 출력단에 배치.
- **Level shifter**: 전압 다른 도메인 간 신호 변환(low↔high). 전압 경계마다 필요.

## 예상 면접 Q&A

### Q1. dynamic power와 static power(leakage)를 구분해 설명하세요.
**A.** dynamic power는 신호가 toggle할 때 캐패시터를 충방전하며 쓰는 전력으로 P = α·C·V²·f, 여기에 입출력이 동시에 켜지는 짧은 순간의 short-circuit power가 더해집니다. static power는 toggle이 없어도 흐르는 leakage(주로 subthreshold + gate tunneling)로 V·I_leak입니다. 미세공정·고온일수록 leakage 비중이 커져 둘 다 잡아야 합니다. V가 제곱으로 들어가는 dynamic 때문에 전압 낮추기가 가장 강력합니다.
- ↳ 꼬리질문: 둘의 비중은 공정에 따라? → 선단 공정(낮은 Vt, 얇은 oxide)일수록 leakage가 커져 idle 전력의 주범. 그래서 power gating이 중요해짐.
- 🧠 메모리 브릿지: DRAM은 refresh·standby leakage가 모바일 배터리 핵심 → LPDDR이 전압·refresh를 공격적으로 낮춤.

### Q2. clock gating의 원리와 ICG 셀이 latch 기반인 이유는?
**A.** 사용하지 않는 레지스터 그룹의 clock을 enable이 0일 때 차단해, clock tree 토글과 FF 내부 switching 전력을 동시에 없앱니다. 단순 AND로 gating하면 enable이 clock high 중에 바뀌며 출력에 glitch(잘못된 clock 펄스)가 생기므로, **ICG는 clock low 동안 enable을 latch로 잡아** clock과 정렬해 glitch 없는 gated clock을 만듭니다. dynamic power 절감의 가장 보편적·효과적 기법입니다.
- ↳ 꼬리질문: gating으로 인한 timing 영향은? → gated clock path가 늘어 clock tree 균형/skew를 봐야 하고, enable 신호의 setup이 추가됨.
- 🧠 메모리 브릿지: 메모리 컨트롤러의 미사용 channel/bank 로직을 clock gating해 활성 전력 절감.

### Q3. power gating의 구조와 함께 필요한 셀들을 설명하세요.
**A.** idle 블록과 전원 사이에 sleep transistor(header=PMOS로 VDD 차단, footer=NMOS로 GND 차단, MTCMOS)를 넣어 전원을 끊어 **leakage까지 제거**합니다. 켜고 끌 때 (1)**isolation cell**로 출력을 0/1 clamp(X 전파 방지), (2)**retention flop**으로 끄기 전 상태 보관, (3)전원 on/off 순서·rush current 제어 시퀀싱, (4)복원 시 retention restore가 필요합니다.
- ↳ 꼬리질문: clock gating과의 차이? → clock gating은 dynamic만 줄이고 전원은 켜둠(leakage 남음). power gating은 leakage까지 없애지만 wake-up latency·면적·복잡도 큼.
- 🧠 메모리 브릿지: deep power-down 모드에서 주변 로직 전원을 끊되 핵심 상태는 retention으로 보존.

### Q4. multi-Vt와 DVFS는 각각 무엇을 줄이나요?
**A.** multi-Vt는 셀의 문턱전압을 골라 쓰는 기법으로, 빠르지만 leaky한 low-Vt를 critical path에만, 느리지만 저leakage인 high-Vt를 여유 path에 배치해 timing을 지키면서 leakage(static)를 줄입니다. DVFS는 워크로드가 가벼울 때 전압과 주파수를 함께 낮춰 dynamic power(∝V²f)를 크게 줄입니다. 전자는 합성/STA가 자동 swap, 후자는 PMU가 런타임 제어합니다.
- ↳ 꼬리질문: DVFS에서 전압을 낮추면 왜 주파수도 낮춰야? → 전압이 낮으면 게이트 지연이 늘어 같은 주파수로는 setup 위반 → V와 f를 짝지어 조절.
- 🧠 메모리 브릿지: 메모리 동작 모드/속도 등급에 따라 전압·주파수를 바꾸는 것도 같은 원리.

### Q5. UPF란 무엇이고 왜 RTL과 분리해서 기술하나요?
**A.** UPF(Unified Power Format, IEEE 1801)는 power domain, supply net, isolation·retention·level shifter, power state table 같은 **전력 의도(power intent)**를 RTL 기능 기술과 분리해 별도 파일로 적는 표준입니다. 같은 RTL을 여러 전력 구조로 구현·검증할 수 있고, 합성·검증·배치 도구가 일관된 power intent를 공유해 isolation 누락 같은 실수를 정적으로 잡을 수 있습니다.
- ↳ 꼬리질문: UPF 없이 RTL에 직접 isolation 넣으면? → 기능과 전력이 섞여 재사용·검증이 어렵고, 도구 간 일관성 깨짐.
- 🧠 메모리 브릿지: 멀티 power domain을 가진 메모리 IP의 전력 구조를 UPF로 기술·검증.

### Q6. retention flop, isolation cell, level shifter의 역할을 각각 설명하세요.
**A.** retention flop은 always-on supply에 연결된 shadow latch(balloon)를 가져 메인 전원이 꺼져도 상태를 보관하고 깨어날 때 복원합니다. isolation cell은 전원이 꺼진 도메인의 출력이 floating/X가 되어 켜진 도메인을 오염시키지 않게 출력을 0 또는 1로 clamp합니다(보통 도메인 출력단). level shifter는 서로 다른 전압의 두 도메인 사이에서 신호 레벨을 변환해, 낮은 전압 신호가 높은 전압 게이트를 제대로 못 켜는(또는 leakage) 문제를 막습니다.
- ↳ 꼬리질문: isolation과 level shifter를 같이 쓰는 순서? → 보통 source 도메인 출력에 isolation, 전압 경계에 level shifter. power state에 따라 isolation enable 제어.
- 🧠 메모리 브릿지: 코어/IO/PHY가 서로 다른 전압을 쓰는 메모리 IP 경계에 level shifter, power-down 경계에 isolation.

### Q7. 저전력 설계에서 dynamic power를 줄이는 우선순위를 어떻게 잡나요?
**A.** P=α·C·V²·f에서 효과 순서로 봅니다. (1)**V**가 제곱이라 전압 island/DVFS로 전압을 낮추는 게 최우선, (2)**α·f**를 clock gating으로 줄여 미사용 로직의 토글 제거, (3)**C**는 좋은 배치·buffer 최적화로 부하 축소, (4)leakage는 multi-Vt·power gating. 보통 clock gating은 거의 공짜라 기본 적용하고, 전압/전원 도메인은 시스템 설계 단계에서 결정합니다.
- ↳ 꼬리질문: glitch power도 dynamic인가? → 그렇다. 불필요한 combinational glitch가 토글로 전력 소모 → 균형 잡힌 로직/path balancing으로 감소.
- 🧠 메모리 브릿지: 메모리는 array 동작(charge/sense)·IO 토글이 큰 전력원 → 전압·활성 bank 최소화가 핵심.

## 30초 암기 요약
- Dynamic = α·C·V²·f (+short-circuit). V 제곱 → 전압 낮추기가 최강.
- Static = leakage(subthreshold 지배+gate). 선단 공정·고온에서 급증.
- Clock gating=미사용 clock 차단(dynamic↓), ICG는 latch로 glitch 방지. 거의 기본 적용.
- Power gating=sleep TR(header/footer, MTCMOS)로 전원 차단 → leakage 제거. isolation+retention+시퀀싱 필요.
- Multi-Vt: low-Vt(빠름·leaky) critical에, high-Vt(느림·저leakage) 여유 path에 → leakage↓.
- DVFS: V·f 동시 하향, P∝V²f. 전압 낮추면 지연 늘어 주파수도 같이 낮춤.
- UPF(IEEE 1801)=power intent를 RTL과 분리 기술(domain/iso/retention/LS).
- retention flop=always-on shadow latch로 state 보존, isolation cell=꺼진 도메인 출력 0/1 clamp, level shifter=전압 경계 변환.
