<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8" />
  <title>자리 뽑기 (X 임의 배치)</title>
  <style>
    body { font-family: sans-serif; }
    .layout {
      display: flex;
      justify-content: center;
      gap: 40px;
      margin-top: 20px;
      flex-wrap: wrap;
    }
    .section {
      display: grid;
      gap: 10px;
    }
    .cell {
      background: #f0f0f0;
      border: 1px solid #aaa;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 18px;
      cursor: pointer;
      width: 60px;
      height: 60px;
      user-select: none;
    }
    .clicked {
      background: #ffcccc;
    }
    .board {
      width: 400px;
      margin: 0 auto;
      height: 40px;
      background: #d0d0d0;
      text-align: center;
      line-height: 40px;
      font-weight: bold;
      margin-bottom: 10px;
    }
    .inputs {
      text-align: center;
      margin-bottom: 10px;
    }
    .unplaced-box {
      max-width: 600px;
      margin: 20px auto;
      background: #ffeeee;
      border: 1px dashed #ff9999;
      padding: 10px;
      font-size: 16px;
      text-align: center;
    }
  </style>
</head>
<body>
  <h2 style="text-align:center;">자리 뽑기</h2>
  <div class="board">(칠판)</div>

  <div class="inputs">
    시작 번호: <input type="number" id="startNum" value="1" />
    끝 번호: <input type="number" id="endNum" value="28" />
    제외할 번호(쉼표로): <input type="text" id="excludeNums" placeholder="예: 3,5,12" />
    <button onclick="generateSeats()">자리 생성</button>
  </div>

  <div class="layout" id="seatLayout"></div>

  <div id="unplaced" class="unplaced-box" style="display:none;"></div>

  <script>
    function shuffle(array) {
      for (let i = array.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [array[i], array[j]] = [array[j], array[i]];
      }
    }

    const seatStates = [];

    function createCell(index, value) {
      const div = document.createElement("div");
      div.className = "cell";
      div.dataset.seatIndex = index;

      if (seatStates[index] === "X") {
        // 기존에 클릭해서 표시한 X는 '×' 표시 유지
        div.textContent = "×";
        div.classList.add("clicked");
      } else {
        // 여기서 value가 'X'인 경우는 빈칸으로 표시
        div.textContent = (value === "X" || value === "") ? "" : value;
        if (value === "X") div.classList.remove("clicked");
      }

      div.addEventListener("click", () => {
        const idx = parseInt(div.dataset.seatIndex);
        if (seatStates[idx] === "X") {
          seatStates[idx] = null;
          div.textContent = (value === "X" || value === "") ? "" : value;
          div.classList.remove("clicked");
        } else {
          seatStates[idx] = "X";
          div.textContent = "×";
          div.classList.add("clicked");
        }
      });

      return div;
    }

    function generateSeats() {
      const start = parseInt(document.getElementById("startNum").value);
      const end = parseInt(document.getElementById("endNum").value);
      const exclude = document.getElementById("excludeNums").value
        .split(",")
        .map(s => parseInt(s.trim()))
        .filter(n => !isNaN(n));

      if (start >= end || start < 1 || end < 1) {
        alert("시작과 끝 번호를 정확히 입력해주세요.");
        return;
      }

      const seatLayout = document.getElementById("seatLayout");
      const unplacedBox = document.getElementById("unplaced");
      seatLayout.innerHTML = "";
      unplacedBox.style.display = "none";
      unplacedBox.innerHTML = "";

      let allNumbers = [];
      for (let i = start; i <= end; i++) {
        allNumbers.push(i);
      }

      // 기존 클릭해서 표시한 X 위치 저장
      const existingXIndices = [];
      for (let i = 0; i < seatStates.length; i++) {
        if (seatStates[i] === "X") existingXIndices.push(i);
      }

      // 제외 번호 필터링
      allNumbers = allNumbers.filter(n => !exclude.includes(n));

      let seatValues = [];

      const totalSeats = end - start + 1;
      if (totalSeats <= 18) {
        // 부족한 빈칸 개수
        const emptyCount = 18 - totalSeats;

        // 먼저 숫자만 섞음
        shuffle(allNumbers);

        // 임의로 빈칸 위치를 만들기 위해 배열에 'X'를 emptyCount 만큼 넣기
        let combined = allNumbers.slice();
        for (let i = 0; i < emptyCount; i++) {
          combined.push("X");
        }
        shuffle(combined);

        seatValues = combined;
      } else {
        // 19 이상이면 그냥 숫자만 셔플하고 제외된 번호는 빈칸 처리
        shuffle(allNumbers);

        // 빈칸은 빈 문자열로 표시
        seatValues = allNumbers.slice();
      }

      // 기존에 '×' 표시가 있던 칸은 유지 (범위 내면)
      existingXIndices.forEach(idx => {
        if (idx < seatValues.length) {
          seatValues[idx] = "X";
          seatStates[idx] = "X";
        }
      });

      // seatStates 배열 크기 맞추고 초기화 (X 아닌 건 null)
      seatStates.length = seatValues.length;
      for (let i = 0; i < seatStates.length; i++) {
        if (seatStates[i] !== "X") seatStates[i] = null;
      }

      let columns = 2,
        rows = 3,
        sections = 3;

      if (totalSeats > 18) {
        const perSection = Math.ceil(seatValues.length / 3);
        rows = Math.ceil(perSection / 2);
      }

      for (let s = 0; s < 3; s++) {
        const section = document.createElement("div");
        section.className = "section";
        section.style.gridTemplateColumns = `repeat(${columns}, 60px)`;
        section.style.gridTemplateRows = `repeat(${rows}, 60px)`;

        for (let row = 0; row < rows; row++) {
          for (let col = 0; col < columns; col++) {
            const localIndex = row * columns + col;
            const globalIndex = s * columns * rows + localIndex;
            const val = seatValues[globalIndex] ?? "";
            section.appendChild(createCell(globalIndex, val));
          }
        }
        seatLayout.appendChild(section);
      }

      const used = 3 * columns * rows;
      const unplaced = seatValues.slice(used).filter(v => v !== "" && v !== "X");
      if (unplaced.length > 0) {
        unplacedBox.style.display = "block";
        unplacedBox.textContent = "배치되지 못한 번호: " + unplaced.join(", ");
      }
    }
  </script>
</body>
</html>
