# concepts/ — 정독용 심층 개념서 (읽는 법)

> **이 폴더의 정체성**: `daily_study/`(암기 카드)·`03_interview_job_qbank.md`(키포인트 사전)와 **다른 종류의 문서**다. 여기는 **"시간 날 때마다 정독하며 개념이 머릿속에 완전히 자리잡게" 하는 교과서/해설서**다. 요약이 아니라 1st-principles로 "왜?"를 끝까지 파고든다.
>
> **언제 읽나**: 출퇴근·자기 전 등 **차분히 읽을 시간**에 한 편씩. 한 번에 다 읽으려 하지 말 것. 한 문서 = 한 주제를 "완전히 이해"하는 한 세션.
>
> **무엇과 연결되나**: 각 문서 끝의 "면접 연결"과 "스스로 점검"은 `06_mock_exam.md`의 문항과 짝이다. **개념서로 이해 → 모의고사로 구술 → 막히면 개념서 재독**의 순환.

---

## 지원 직무 = 설계(Design). 그래서 이 순서로 읽어라.

JD 기준 핵심은 **디지털설계·RTL·SoC·검증·DFT·STA·CDC/RDC·SI/PI·고속 인터페이스**다. 공정·소자물성·양산은 별도 직무군이므로 라이트(11)로만 둔다.

### 🥇 1순위 — 지원자 강점이자 직무 핵심 (먼저, 깊게)
| 문서 | 주제 | 왜 1순위 |
|---|---|---|
| `05_CDC_RDC_완전이해.md` | 클럭/리셋 도메인 크로싱, 메타스터빌리티, 동기화 | 지원자 **최대 강점** + 메모리 Peri 다중도메인 핵심 |
| `06_STA_완전이해.md` | Static Timing Analysis, setup/hold, 코너, closure | 지원자 강점 + HBM/DDR 고속 I/O 마진 직결 |
| `07_검증_DFT_MBIST_완전이해.md` | UVM·coverage·정합성·scan/ATPG·MBIST/repair | 지원자 강점 + JD "검증" 직무 정조준 |
| `12_RTL_디지털설계_기초.md` | 조합/순차·FSM·파이프라인·blocking/non-blocking·reset 전략·합성·빌딩블록 | **"직접 RTL 설계 가능"** 갭 방어 + 모든 설계롤 토대 |
| `13_온칩버스_인터커넥트.md` | valid/ready·AMBA AXI/AHB/APB·arbitration·NoC·async bridge | JD 명시(AMBA/NoC/Top Bus/Async Bridge), SoC 통합 |

### 🥈 2순위 — 메모리 디바이스·아키텍처 갭 보강 (반드시)
| 문서 | 주제 | 비고 |
|---|---|---|
| `02_DRAM_완전이해.md` | 1T1C·sensing·타이밍·refresh·bank·Peri·스케일링 | ★기출 다수("DRAM 설명", "refresh 왜") |
| `14_메모리컨트롤러_RAS.md` | 명령/스케줄링·refresh관리·ODT·JEDEC·ECC/scrubbing/RAS | JD 명시("DRAM Memory Controller", "RAS") — 메모리의 디지털 두뇌 |
| `04_HBM_패키징_완전이해.md` | 메모리 월·TSV·적층·MR-MUF·base die·수율·SI/PI | ★기출("HBM 시장", "SI 대응") + SK 핵심 제품 |
| `09_고속인터페이스_PHY_SIPI_완전이해.md` | 고속 I/O·등화(DFE/CTLE/FFE)·DLL/PLL·SI/PI | JD 최빈출 키워드(RX/TX/ODT/Equalizer/SerDes) |
| `03_NAND_완전이해.md` | 전하저장·string·program/erase·3D·수명·레이턴시/쓰루풋 | ★기출("NAND 설명", "DRAM vs NAND") |

### 🥉 3순위 — 기초·신뢰성·산업 (강점/갭을 받쳐주는 토대)
| 문서 | 주제 | 비고 |
|---|---|---|
| `01_소자_MOSFET_완전이해.md` | 반도체·MOSFET·동작영역·CMOS·SCE·leakage·FinFET/GAA | ★기출("MOSFET 동작", "SCE") + STA/DRAM의 물리 뿌리 |
| `08_저전력_신뢰성_완전이해.md` | 전력성분·clock/power gating·retention·RowHammer·ECC·aging | clock gating은 STA/CDC 직결 |
| `10_산업_AI메모리_PIM_완전이해.md` | HBM 시장·HBM4·PIM·CXL·인터커넥트 | ★기출("HBM 위상") — 단 수치는 변동, 면접 직전 최신화 |

### 📎 참고 (우선순위 낮음)
| 문서 | 주제 | 비고 |
|---|---|---|
| `11_공정_라이트.md` | 8대공정·포토·식각 (교양 수준) | **별도 직무군**. "모른다" 안 들릴 정도만. 마지막에. |

---

## 2주 정독 플랜 (하루 1편, 모의고사와 병행)

> `daily_study/_schedule.md`의 암기 루틴 위에 "정독 1편"을 얹는다. 정독 후 `06_mock_exam.md`의 해당 🔗 문항을 즉시 구술.

| Day | 정독 | 직후 구술(모의고사) |
|---|---|---|
| D-14 | 05 CDC/RDC | 기술 SET A: A1·A2·A9 |
| D-13 | 06 STA | 기술 SET A: A3 / a!sk SET3 Q4 |
| D-13 | 12 RTL 설계 기초 | 기술 SET C: C1~C4 / PART4: P1~P4 |
| D-12 | 07 검증/DFT/MBIST | 기술 SET A: A4·A5·A6·A7 |
| D-12 | 13 온칩버스 | 기술 SET C: C5~C7·C10 / PART4: P5~P9 |
| D-11 | 02 DRAM | 기술 SET B: B1·B2 / a!sk SET1 Q4 |
| D-11 | 14 메모리컨트롤러/RAS | 기술 SET C: C8·C9 / 드릴 24·26 |
| D-10 | 04 HBM | 기술 SET B: B4·B9 |
| D-9 | 09 고속I/O·SI/PI | 기술 SET B: B5 / 빠른드릴 7·9·10·20 |
| D-8 | 03 NAND | 기술 SET B: B3·B1 / 드릴 21·22 |
| D-7 | 01 소자/MOSFET | 기술 SET B: B6·B7 / 드릴 23 |
| D-6 | 08 저전력/신뢰성 | 기술 SET A: A8 / B: B8 |
| D-5 | 10 산업(+최신화) | 기술 SET B: B9·B10 |
| D-4~ | 약한 문서 재독 | a!sk SET1~3 통째 녹화 |
| 직전 | 11 공정 라이트 훑기 | 점수 낮은 문항만 |

> 14편이라 하루 1편이 빠듯하면, 강점 영역(05·06·07·12·13)을 먼저 빠르게 1회독하고 약한 곳을 2회독하는 식으로 우선순위를 둔다. 핵심은 "정독 후 즉시 구술"의 짝 맞춤.

---

## 읽을 때의 자세 (중요)

1. **수동적으로 읽지 말 것.** 각 "왜?"에서 책을 덮고 스스로 답해본 뒤 본문과 비교.
2. **그림을 직접 그려라.** sense amp, 2-FF 동기화기, setup/hold 파형은 손으로 그릴 수 있어야 "이해"한 것.
3. **끝의 "스스로 점검"을 입으로 답하라.** 막히는 항목 = 다시 읽을 곳.
4. **항상 브릿지로 착지.** "내가 로직에서 한 X = 메모리의 Y". 이게 면접의 승부처다.
5. 한 번에 완벽히 이해 못 해도 된다. **2주에 걸쳐 2~3회 회독**하면 자리잡는다.

> 모든 수치 중 시장·제품 관련(점유율·노드·속도)은 시즌마다 바뀐다. 본문의 **(변동/추정)** 표기를 신뢰하고, 면접 직전 IR/뉴스로 최신화(특히 `10`).
