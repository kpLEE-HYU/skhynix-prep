# C_NAND — 3D NAND Flash

> 📌 floating gate/charge trap에 F-N 터널링으로 전하를 넣고 빼 비휘발성으로 저장하고, 셀을 수직 적층(300단+)해 집적하는 메모리 · 🏷️ P2 · 🧠 ECC·wear leveling·read disturb는 controller(FTL) 안의 디지털 알고리즘 — 소자의 비신뢰성을 로직으로 덮는 설계

## 핵심 개념
- **저장 방식**: transistor의 게이트 아래 전하저장층에 전자를 가둬 **문턱전압(Vt) 이동**으로 0/1 저장. 전원 꺼져도 유지=비휘발성.
- **FG vs CTF**: **Floating Gate**=도체 poly에 전하 저장(연속). **Charge Trap(CTF)**=절연체 Si₃N₄ trap에 전하를 이산적으로 포획 → cell-to-cell 간섭↓, 단일 결함에 강함, **3D 적층에 유리** → 현대 3D NAND는 CTF.
- **F-N 터널링**: 고전계로 얇은 oxide를 전자가 터널링. **Program**=전하 주입(Vt↑), **Erase**=전하 제거(Vt↓).
- **단위**: **read/program=page 단위**, **erase=block 단위**. 제자리 덮어쓰기 불가 → 새 page에 쓰고 옛 block은 통째 erase.
- **P/E cycle 수명**: 터널링이 oxide를 열화 → 횟수 제한. **SLC ~100k, MLC ~3k~10k, TLC ~1k~3k, QLC ~수백~1k**.
- **bit/cell**: **SLC/MLC/TLC/QLC = 1/2/3/4 bit** = 2/4/8/16 Vt 레벨. bit↑ → 밀도·저가↑, but 마진·속도·수명↓, **ECC 부담↑**.
- **3D NAND**: 평면 미세화 한계 → 셀을 **수직 적층**. 적층 후 **channel hole**을 위→아래로 etch, 그 벽에 cell. **string**=수직으로 직렬 연결된 셀 묶음. 현재 **SK하이닉스 321단 양산(2025)**, 삼성 286단(9세대), **400단+ 개발**(SK하이닉스 2025 말, 삼성 10세대 V-NAND 400+). 고단수는 **hybrid bonding**(peri를 별도 wafer로 만들어 본딩, CMOS-bonded-array).
- **Wear leveling**: FTL이 write를 전체 block에 고르게 분산 → 특정 block 조기 마모 방지.
- **ECC**: P/E·retention·disturb로 늘어나는 bit 오류를 강력한 **LDPC** ECC로 정정.
- **Read disturb**: 한 page를 읽을 때 같은 string의 비선택 셀에 pass 전압(Vpass)을 거는데, 반복되면 그 셀에 약한 전하 주입 → 여러 번 read 후 bit flip. ECC + read-disturb-aware refresh로 완화.

## 예상 면접 Q&A
### Q1. Floating gate와 charge trap의 차이는? 왜 3D는 CTF인가?
**A.** 둘 다 게이트 아래 전하저장층에 전자를 가둬 Vt를 옮기는 원리지만 저장층 재료가 다릅니다. **Floating Gate**는 도체 poly여서 전하가 층 전체에 연속적으로 퍼지고, 인접 셀과 결합(간섭)이 크며 한 군데 oxide 결함이 생기면 저장 전하가 다 새어나갑니다. **Charge Trap(CTF)**는 절연체 Si₃N₄에 전하를 **이산적인 trap에 포획**해서, cell-to-cell 간섭이 작고 국소 결함에 강합니다. 이 견고함과 단순한 구조 덕에 수직 적층(3D)에 유리해 현대 3D NAND는 대부분 CTF를 씁니다.
- ↳ 꼬리질문: CTF의 단점? → trap 전하의 시간적 산포(retention) 관리, 프로그램 산포 제어가 과제.
- 🧠 설계 브릿지: 소자 단의 "데이터를 어떻게 견고히 가두나"가 곧 위쪽 ECC 강도·refresh 정책을 정한다 — 소자 신뢰성과 컨트롤러 알고리즘은 한 쌍.

### Q2. Program과 erase는 어떻게 일어나고, 왜 단위가 다른가?
**A.** 둘 다 **F-N 터널링**으로 얇은 oxide를 통해 전자를 움직입니다. Program은 셀에 전자를 주입해 Vt를 올리고(보통 page 단위), Erase는 전자를 빼내 Vt를 내립니다(block 단위). **NAND는 구조상 program은 page 단위로 가능하지만 erase는 block 단위로만** 됩니다. 그래서 이미 쓴 자리를 바로 덮어쓸 수 없고, 새 page에 쓴 뒤 옛 데이터는 block 전체를 erase할 때 회수합니다. 이 비대칭이 FTL(garbage collection, wear leveling)이 필요한 근본 이유입니다.
- ↳ 꼬리질문: 제자리 덮어쓰기가 안 되는 게 시스템에 주는 영향? → write amplification, garbage collection 오버헤드, 그래서 over-provisioning과 wear leveling이 필요.
- 🧠 설계 브릿지: erase=block / program=page라는 하드웨어 제약을 FTL이라는 디지털 추상화로 덮는 것 — 하드웨어 한계를 펌웨어/로직으로 가리는 전형적 설계.

### Q3. SLC/MLC/TLC/QLC의 trade-off와 P/E 수명은?
**A.** 셀당 저장 bit 수입니다 — **SLC 1bit(2레벨), MLC 2bit(4), TLC 3bit(8), QLC 4bit(16레벨)**. bit를 늘릴수록 같은 셀에 더 많은 Vt 레벨을 욱여넣어 밀도·원가는 좋아지지만, 레벨 간 전압 마진이 좁아져 **속도·신뢰성·수명이 떨어지고 ECC 부담이 커집니다.** P/E 내구는 대략 **SLC ~10만, MLC ~3천~1만, TLC ~1천~3천, QLC ~수백~1천** 회입니다. 터널링이 oxide를 누적 열화시키기 때문입니다.
- ↳ 꼬리질문: QLC를 어디에 쓰나? → 읽기 위주·대용량 저가 저장(아카이브, read-intensive). 쓰기 많은 곳엔 부적합.
- 🧠 설계 브릿지: bit/cell↑ → 좁은 마진 → 강한 ECC(LDPC) 필요. 소자에서 잃은 신뢰성을 코드(디지털)로 되찾는 구조.

### Q4. 3D NAND는 어떻게 만들고 string이 뭔가?
**A.** 평면(2D) NAND는 미세화 한계에 닿아, 셀을 **수직으로 쌓는 3D 구조**로 전환했습니다. 여러 게이트 층을 먼저 적층한 뒤 위에서 아래로 **channel hole**을 깊게 etch하고, 그 구멍 벽을 따라 CTF 셀이 형성됩니다. 이렇게 한 구멍에 수직으로 직렬 연결된 셀들의 묶음이 **string**입니다. 현재 **SK하이닉스가 321단을 양산(2025)**하고 삼성 286단, **400단+가 개발 중**입니다. 단수가 높아지면 깊은 구멍을 균일하게 뚫는 고종횡비 etch가 어려워, peri를 별도 wafer로 만들어 붙이는 **hybrid bonding**을 씁니다.
- ↳ 꼬리질문: 단수를 무한정 못 올리는 이유? → channel hole의 고종횡비 etch 한계, 적층 응력·휨(warpage), string 저항 증가. 그래서 string stacking으로 나눠 쌓는다.
- 🧠 설계 브릿지: NAND는 "면적을 줄이지 않고 Z축으로 쌓는다"는 점에서 DRAM 미세화와 정반대 — 같은 집적도를 다른 물리 축으로 푸는 설계 선택.

### Q5. Read disturb가 뭐고 어떻게 막나?
**A.** NAND는 한 page를 읽을 때 선택 셀만 정확히 읽기 위해, **같은 string의 비선택 셀에는 무조건 켜지도록 pass 전압(Vpass)을 겁니다.** 이 Vpass가 비선택 셀에 약한 전하 주입을 일으켜, 같은 영역을 **수없이 반복해서 읽으면** 그 셀들의 Vt가 조금씩 올라가 결국 bit가 뒤집힐 수 있습니다. 이것이 read disturb입니다. 대응은 **강력한 ECC로 정정**하고, 특정 block의 누적 read 횟수를 세어 임계 넘으면 데이터를 다른 곳으로 옮겨쓰는 **read-disturb-aware refresh(read reclaim)**를 컨트롤러가 수행하는 것입니다.
- ↳ 꼬리질문: program disturb와 차이? → program disturb는 쓰기 중 인접 셀 교란, read disturb는 읽기 반복 누적 교란. 둘 다 ECC·관리로 완화.
- 🧠 설계 브릿지: read disturb 카운팅·재배치는 컨트롤러 FSM/카운터·정책 로직 — 소자의 약점을 디지털 제어로 감시·교정하는 일.

### Q6. NAND의 신뢰성을 컨트롤러는 어떻게 보장하나? (wear leveling·ECC)
**A.** NAND 셀은 본질적으로 신뢰도가 낮고(수명 제한·disturb·retention 저하) 그대로는 못 쓰므로, **컨트롤러(FTL)가 디지털 알고리즘으로 덮습니다.** ① **Wear leveling**: write를 전체 block에 고르게 분산해 특정 block이 먼저 마모되는 걸 막습니다. ② **ECC**: P/E·retention·disturb로 늘어나는 bit 오류를 **LDPC** 같은 강력한 코드로 정정합니다. ③ 여기에 garbage collection, read reclaim, bad block 관리가 더해집니다. 즉 소자의 비신뢰성을 **로직·코드로 신뢰성 있는 저장장치로 변환**하는 것이 NAND 설계의 핵심입니다.
- ↳ 꼬리질문: bit/cell이 늘수록(QLC) 컨트롤러 부담은? → 좁은 마진으로 raw BER이 높아져 더 강한 LDPC, 더 정교한 read-retry/voltage tracking이 필요.
- 🧠 설계 브릿지: ECC 인코더/디코더, wear leveling, GC는 전형적 디지털 RTL 블록 — 제 검증/sign-off 경험이 그대로 적용되는 controller SoC 영역.

## 30초 암기 요약
- 저장=전하저장층의 **Vt 이동**, 비휘발성. **FG(도체) vs CTF(절연체 trap)** → 3D는 간섭↓·견고한 **CTF**.
- **F-N 터널링**으로 program(Vt↑)/erase(Vt↓). **read·program=page, erase=block** → 제자리 덮어쓰기 불가.
- bit/cell: **SLC1 / MLC2 / TLC3 / QLC4**. P/E 수명 **SLC~10만, TLC~1k~3k, QLC~수백**.
- 3D NAND=수직 적층 + **channel hole etch**, **string**=수직 직렬 셀. **SK하이닉스 321단 양산(2025)**, 400단+ 개발, 고단수=hybrid bonding.
- **Read disturb**=반복 읽기 시 Vpass로 비선택 셀 교란 → ECC + read reclaim으로 완화.
- 컨트롤러(FTL)가 **wear leveling + LDPC ECC + GC**로 소자 비신뢰성을 덮음 = 디지털 설계 영역.
