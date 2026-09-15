# 2주차 과제 — 지구·달·인공위성의 변환 설계

**과목:** 컴퓨터그래픽스  
**이름:** 유락진  
**학번:** 202421936

---

## 조사한 실제 수치

| 천체 | 반지름 | 지구와의 거리 | 출처 |
|---|---|---|---|
| 지구 | 6,371 km | — | NASA Planetary Fact Sheet |
| 달 | 1,737 km | 384,400 km | NASA Planetary Fact Sheet |
| ISS (국제우주정거장) | — (크기 약 109 m) | 고도 400 km (궤도 반지름 약 6,771 km) | NASA ISS Facts |

**거리 단위:** 지구 반지름 = 1 로 정했다.  
이유: 지구–달 거리가 지구 반지름의 60.3배로 숫자가 적당히 크고, float32의 유효숫자 7자리를 넘지 않아 정밀도 문제가 없다. 만약 미터 단위를 쓰면 지구–달 거리가 384,400,000 이 되어 ISS의 109 m 같은 작은 값이 반올림으로 사라진다.

---

## Task 1 — 실제 비율로 만들기

### 변환 입력값

**지구**
- `Rz(t*10)` — 자전

**달** (왼쪽부터 오른쪽 순서)
- `Rz(t*5)` — 공전 (조석 고정: 한쪽 면이 계속 지구를 향함)
- `T(60.3, 0, 0)` — 지구로부터의 거리 (지구 반지름 기준)
- `S(0.273, 0.273, 0.273)` — 달의 반지름 (지구 대비 0.273배)

**인공위성 (ISS)**
- `Rz(t*60)` — 공전 (약 90분 주기)
- `T(1.063, 0, 0)` — 궤도 반지름 (지구 반지름 + 고도 400km = 1.063)
- `S(0.0000171, 0.0000171, 0.0000171)` — ISS 크기 (109m / 6,371km ≈ 0.0000171)

**축 범위:** x, y, z 모두 ±70

### 공유 링크

- [Task 1 실행하기](https://cg.catholic.ac.kr/~mgchoi/CG/demos/d02-transform-lab.html?d=eyJyYW5nZSI6eyJ4IjoiNzAiLCJ5IjoiNzAiLCJ6IjoiNzAifSwib2JqZWN0cyI6W3siaWQiOiJlYXJ0aCIsIm5hbWUiOiLsp4DqtawiLCJjb2xvciI6WzAuMzUsMC42LDAuOTVdLCJzdGVwcyI6W3sidHlwZSI6IlJ6IiwiYXJncyI6WyJ0KjEwIl19XX0seyJpZCI6Im1vb24iLCJuYW1lIjoi64usIiwiY29sb3IiOlswLjc4LDAuNzgsMC44Ml0sInN0ZXBzIjpbeyJ0eXBlIjoiUnoiLCJhcmdzIjpbInQqNSJdfSx7InR5cGUiOiJUIiwiYXJncyI6WyI2MC4zIiwiMCIsIjAiXX0seyJ0eXBlIjoiUyIsImFyZ3MiOlsiMC4yNzMiLCIwLjI3MyIsIjAuMjczIl19XX0seyJpZCI6InNhdCIsIm5hbWUiOiLsnbjqs7XsnITshLEiLCJjb2xvciI6WzAuOTUsMC43MiwwLjM1XSwic3RlcHMiOlt7InR5cGUiOiJSeiIsImFyZ3MiOlsidCo2MCJdfSx7InR5cGUiOiJUIiwiYXJncyI6WyIxLjA2MyIsIjAiLCIwIl19LHsidHlwZSI6IlMiLCJhcmdzIjpbIjAuMDAwMDE3MSIsIjAuMDAwMDE3MSIsIjAuMDAwMDE3MSJdfV19XX0=)

### 답변

1. **거리의 단위를 무엇으로 정했는가? 왜 그렇게 정했는가?**  
   지구 반지름을 1로 정했다. 지구–달 거리가 60.3배로 숫자가 적당하고, float32 유효숫자(7자리)를 넘지 않아 정밀도 문제가 없다.

2. **숫자가 커서 생긴 문제가 있었는가?**  
   미터 단위를 썼다면 지구–달 거리(384,400,000m)가 7자리를 넘어 ISS의 109m가 반올림으로 사라졌을 것이다. 지구 반지름 단위를 써서 이 문제를 피했다.

3. **달·위성이 지구를 향하게 만든 것은 어느 변환 단계 덕분인가?**  
   가장 왼쪽(마지막에 적용되는)의 `Rz` 회전 덕분이다. `Rz · T · S` 순서에서 정점은 먼저 S로 크기가 정해지고 T로 +X 방향으로 밀려난 뒤, 마지막에 Rz로 통째로 회전한다. 이때 물체의 로컬 좌표축도 함께 회전하므로 +X 방향이 계속 지구 중심을 가리키게 된다 (조석 고정).

![Task 1 — 실제 비율로 배치한 지구와 인공위성](images/task1.png)

---

## Task 2 — NDC 범위에 맞추기

### 변환 입력값

모든 물체의 행렬 사슬 **가장 앞(가장 왼쪽)**에 공통 배율 `S(0.015)` 를 추가했다.

**지구**
- `S(0.015, 0.015, 0.015)` — 전체 축소
- `Rz(t*10)` — 자전

**달**
- `S(0.015, 0.015, 0.015)` — 전체 축소
- `Rz(t*5)` — 공전
- `T(60.3, 0, 0)` — 거리
- `S(0.273, 0.273, 0.273)` — 달 크기

**인공위성 (ISS)**
- `S(0.015, 0.015, 0.015)` — 전체 축소
- `Rz(t*60)` — 공전
- `T(1.063, 0, 0)` — 궤도 반지름
- `S(0.0000171, 0.0000171, 0.0000171)` — ISS 크기

**축 범위:** x, y, z 모두 ±1 (NDC)

### 공유 링크

- [Task 2 실행하기](https://cg.catholic.ac.kr/~mgchoi/CG/demos/d02-transform-lab.html?d=eyJyYW5nZSI6eyJ4IjoiMSIsInkiOiIxIiwieiI6IjEifSwib2JqZWN0cyI6W3siaWQiOiJlYXJ0aCIsIm5hbWUiOiLsp4DqtawiLCJjb2xvciI6WzAuMzUsMC42LDAuOTVdLCJzdGVwcyI6W3sidHlwZSI6IlMiLCJhcmdzIjpbIjAuMDE1IiwiMC4wMTUiLCIwLjAxNSJdfSx7InR5cGUiOiJSeiIsImFyZ3MiOlsidCoxMCJdfV19LHsiaWQiOiJtb29uIiwibmFtZSI6IuuLrCIsImNvbG9yIjpbMC43OCwwLjc4LDAuODJdLCJzdGVwcyI6W3sidHlwZSI6IlMiLCJhcmdzIjpbIjAuMDE1IiwiMC4wMTUiLCIwLjAxNSJdfSx7InR5cGUiOiJSeiIsImFyZ3MiOlsidCo1Il19LHsidHlwZSI6IlQiLCJhcmdzIjpbIjYwLjMiLCIwIiwiMCJdfSx7InR5cGUiOiJTIiwiYXJncyI6WyIwLjI3MyIsIjAuMjczIiwiMC4yNzMiXX1dfSx7ImlkIjoic2F0IiwibmFtZSI6IuyduOqzteychOyEsSIsImNvbG9yIjpbMC45NSwwLjcyLDAuMzVdLCJzdGVwcyI6W3sidHlwZSI6IlMiLCJhcmdzIjpbIjAuMDE1IiwiMC4wMTUiLCIwLjAxNSJdfSx7InR5cGUiOiJSeiIsImFyZ3MiOlsidCo2MCJdfSx7InR5cGUiOiJUIiwiYXJncyI6WyIxLjA2MyIsIjAiLCIwIl19LHsidHlwZSI6IlMiLCJhcmdzIjpbIjAuMDAwMDE3MSIsIjAuMDAwMDE3MSIsIjAuMDAwMDE3MSJdfV19XX0=)

### 답변

1. **s 를 얼마로 정했고 그 값을 어떻게 계산했는가?**  
   s = 0.015 로 정했다. 가장 먼 지점은 달의 바깥쪽 가장자리로, 지구–달 거리 60.3 + 달 반지름 0.273 = 60.57 이다. s = 1/60.57 ≈ 0.0165 이고, 가장자리가 잘리지 않도록 여유를 두어 0.015 로 정했다.

2. **배율 행렬을 사슬의 맨 앞에 넣은 이유는 무엇인가? 맨 뒤에 넣으면 어떻게 되는가?**  
   행렬은 오른쪽부터 정점에 적용된다. 맨 앞(가장 왼쪽)에 넣으면 가장 마지막에 적용되어, 이미 제자리에 놓인 물체의 **위치와 크기를 모두** 축소한다. 맨 뒤(가장 오른쪽)에 넣으면 가장 먼저 적용되어 크기만 줄어들고 위치는 그대로이므로, 달이 여전히 60.3 만큼 떨어져 있어 화면 밖으로 나간다.

3. **세 물체에 같은 배율을 쓴 이유는 무엇인가?**  
   실제 비율을 유지한 채 장면 전체를 NDC 안에 넣기 위해서이다. 물체마다 다른 배율을 쓰면 상대적 크기 비율이 깨진다.

4. **비율을 유지한 결과, 화면에서 지구와 인공위성은 어떻게 보이는가?**  
   지구는 반지름이 0.015 로 아주 작은 점으로 보이고, 인공위성(ISS)은 0.015 × 0.0000171 ≈ 2.6×10⁻⁷ 로 너무 작아 화면에서 완전히 보이지 않는다. 달도 0.015 × 0.273 ≈ 0.004 로 겨우 보이는 수준이다.

![Task 2 — NDC 범위에 맞춘 장면](images/task2.png)

---

## Task 3 — 보는 사람을 위한 표현

### 제안한 방법: 크기 과장 (distance는 실제 비율 유지, 크기만 확대)

실제 비율 그대로는 인공위성이 보이지 않고 달도 점에 불과하여, 궤도 구조와 상대적 위치를 이해하기 어렵다. 이를 해결하기 위해 **거리는 실제 비율을 유지하고 물체의 크기만 과장**해서 표현했다.

- 달 크기: 0.273 → **3** (약 11배 확대)
- ISS 크기: 0.0000171 → **0.05** (약 3,000배 확대)

### 변환 입력값

**지구**
- `Rz(t*10)` — 자전

**달**
- `Rz(t*5)` — 공전
- `T(60.3, 0, 0)` — 실제 거리 유지
- `S(3, 3, 3)` — 크기 과장 (11배)

**인공위성 (ISS)**
- `Rz(t*60)` — 공전
- `T(1.063, 0, 0)` — 실제 궤도 반지름 유지
- `S(0.05, 0.05, 0.05)` — 크기 과장 (3,000배)

**축 범위:** x, y, z 모두 ±70

### 공유 링크

- [Task 3 실행하기](https://cg.catholic.ac.kr/~mgchoi/CG/demos/d02-transform-lab.html?d=eyJyYW5nZSI6eyJ4IjoiNzAiLCJ5IjoiNzAiLCJ6IjoiNzAifSwib2JqZWN0cyI6W3siaWQiOiJlYXJ0aCIsIm5hbWUiOiLsp4DqtawiLCJjb2xvciI6WzAuMzUsMC42LDAuOTVdLCJzdGVwcyI6W3sidHlwZSI6IlJ6IiwiYXJncyI6WyJ0KjEwIl19XX0seyJpZCI6Im1vb24iLCJuYW1lIjoi64usIiwiY29sb3IiOlswLjc4LDAuNzgsMC44Ml0sInN0ZXBzIjpbeyJ0eXBlIjoiUnoiLCJhcmdzIjpbInQqNSJdfSx7InR5cGUiOiJUIiwiYXJncyI6WyI2MC4zIiwiMCIsIjAiXX0seyJ0eXBlIjoiUyIsImFyZ3MiOlsiMyIsIjMiLCIzIl19XX0seyJpZCI6InNhdCIsIm5hbWUiOiLsnbjqs7XsnITshLEiLCJjb2xvciI6WzAuOTUsMC43MiwwLjM1XSwic3RlcHMiOlt7InR5cGUiOiJSeiIsImFyZ3MiOlsidCo2MCJdfSx7InR5cGUiOiJUIiwiYXJncyI6WyIxLjA2MyIsIjAiLCIwIl19LHsidHlwZSI6IlMiLCJhcmdzIjpbIjAuMDUiLCIwLjA1IiwiMC4wNSJdfV19XX0=)

### 답변

1. **실제 비율이 정보를 전달하기에 적합한가?**  
   적합하지 않다. Task 1과 Task 2에서 확인했듯, 실제 비율로는 인공위성이 완전히 보이지 않고 달도 작은 점에 불과하다. 시청자는 세 물체의 궤도 관계와 상대적 위치를 파악하기 어렵다.

2. **제안한 방법: 크기 과장**  
   거리(궤도 반지름)는 실제 값을 유지하고, 물체의 시각적 크기만 과장했다. 달은 11배, ISS는 3,000배 확대하여 세 물체가 모두 명확하게 보이도록 했다.

3. **장점과 잃는 것**  
   - **장점:** 세 천체가 모두 화면에서 명확하게 보여, 궤도 구조와 상대적 위치를 한눈에 이해할 수 있다. 교육적·시각적 전달력이 크게 향상된다.  
   - **잃는 것:** 크기 비율이 더 이상 실제가 아니므로, 시청자가 천체의 실제 크기를 오해할 수 있다. 달이 지구의 절반 정도로 보이지만 실제로는 27%에 불과하고, ISS가 눈에 보일 정도의 크기로 보이지만 실제로는 109m에 불과하다는 점이 왜곡된다.

![Task 3 — 크기를 과장하여 표현한 지구·달·위성](images/task3.png)

---

## 실행 코드

- [Task 1 실행하기](task1.html)
- [Task 2 실행하기](task2.html)
- [Task 3 실행하기](task3.html)

*(GitHub Pages 에서 정상적으로 실행되는지 확인)*

---

## 점검표

- [x] 저장소에 `week2/week2.md` 가 있다
- [x] 세 Task 의 변환 입력값이 모두 적혀 있고, 각각 왜 그 값인지 설명이 있다
- [x] Task 별 공유 링크가 실제로 열리고 그 설정이 나온다
- [ ] 중요한 결과의 캡처 이미지가 GitHub 문서에서 깨지지 않고 보인다
- [ ] `task1.html`, `task2.html`, `task3.html` 이 저장소에 있고, Pages 주소로 열면 실제로 움직인다
- [x] 달과 인공위성이 항상 지구를 향한다 (화살표로 확인)
- [x] 조사한 수치의 출처가 적혀 있다
- [x] 단위를 그렇게 정한 이유를 설명할 수 있다
- [x] Task 3 에서 제안한 표현을 실제로 만들어 캡처했다
