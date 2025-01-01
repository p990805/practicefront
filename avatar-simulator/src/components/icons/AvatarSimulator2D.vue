<template>
  <div>
    <h1>2D Avatar Color Filler (8방향 테스트)</h1>
    <!-- 캔버스 영역 -->
    <canvas
      ref="avatarCanvas"
      width="300"
      height="400"
      class="avatar-canvas"
      @click="fillColor"
    ></canvas>

    <!-- 기본 색상 목록 & 지우개 -->
    <div class="color-picker">
      <div
        v-for="(color, idx) in colors"
        :key="idx"
        :style="{ backgroundColor: color }"
        class="color-box"
        @click="selectColor(color)"
      ></div>
      <div class="color-box eraser" @click="useEraser" title="Eraser">🧹</div>
    </div>

    <!-- 커스텀 RGB 입력 -->
    <div class="custom-color">
      <label>
        Custom RGB:
        <input type="number" v-model.number="customColor.r" min="0" max="255" placeholder="R" />
        <input type="number" v-model.number="customColor.g" min="0" max="255" placeholder="G" />
        <input type="number" v-model.number="customColor.b" min="0" max="255" placeholder="B" />
      </label>
      <div
        class="color-preview"
        :style="{ backgroundColor: `rgb(${customColor.r}, ${customColor.g}, ${customColor.b})` }"
      ></div>
      <button @click="addCustomColor">Add Color</button>
    </div>

    <!-- 오차 범위 설정 -->
    <label>
      Tolerance:
      <input type="number" v-model.number="tolerance" min="0" max="255" />
    </label>

    <!-- 리셋 버튼 -->
    <button @click="resetCanvas">Reset</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      selectedColor: "red",       // 선택된 색상
      colors: ["red", "blue", "green", "yellow", "pink", "orange"], // 기본 색상 목록
      customColor: { r: 255, g: 255, b: 255 }, // 사용자가 지정할 RGB 값
      canvas: null,
      ctx: null,
      originalImageData: null,   // 최초 이미지 데이터(Reset 시 복원)
      isEraser: false,           // 지우개 모드인지 여부
      tolerance: 20,            // 오차 범위(기본값)
    };
  },
  methods: {
    // 초기 캔버스 세팅
    initCanvas() {
      this.canvas = this.$refs.avatarCanvas;
      this.ctx = this.canvas.getContext("2d", { willReadFrequently: true });

      const img = new Image();
      // 원본 이미지 경로 (테스트하려면 적절한 이미지로 바꾸세요)
      img.src = "/2d.jpg"; 
      img.onload = () => {
        this.ctx.drawImage(img, 0, 0, this.canvas.width, this.canvas.height);
        this.originalImageData = this.ctx.getImageData(
          0,
          0,
          this.canvas.width,
          this.canvas.height
        );
      };
    },
    // 색상 선택
    selectColor(color) {
      this.selectedColor = color;
      this.isEraser = false;
    },
    // 지우개 선택
    useEraser() {
      this.isEraser = true;
    },
    // 클릭 시 Flood Fill 실행
    fillColor(event) {
      const x = event.offsetX;
      const y = event.offsetY;

      // 현재 캔버스 데이터를 가져옴
      const canvasData = this.ctx.getImageData(0, 0, this.canvas.width, this.canvas.height);
      const data = canvasData.data;

      // 클릭한 픽셀(목표 색상)
      const targetColor = this.getPixel(data, x, y);

      // 지우개 모드면 원본 이미지의 해당 픽셀 색으로 채움
      const fillColor = this.isEraser
        ? this.getPixel(this.originalImageData.data, x, y)
        : this.hexToRgb(this.selectedColor);

      // 이미 같은 색(또는 tolerance 이내)이면 진행할 필요 없음
      if (this.colorsMatch(targetColor, fillColor, this.tolerance)) {
        return;
      }

      // Flood Fill 실행 (8방향)
      this.floodFill(data, x, y, targetColor, fillColor);

      // 바뀐 데이터 다시 캔버스에 반영
      this.ctx.putImageData(canvasData, 0, 0);
    },
    /**
     * 8방향 Flood Fill
     */
    floodFill(data, x, y, targetColor, fillColor) {
      const stack = [[x, y]];
      const width = this.canvas.width;
      const height = this.canvas.height;

      // 8방향(상, 하, 좌, 우, 대각선) 좌표 변위
      const directions = [
        [0, -1],   // 위
        [0, 1],    // 아래
        [-1, 0],   // 왼쪽
        [1, 0],    // 오른쪽
        [-1, -1],  // 왼쪽 위 대각선
        [1, -1],   // 오른쪽 위 대각선
        [-1, 1],   // 왼쪽 아래 대각선
        [1, 1],    // 오른쪽 아래 대각선
      ];

      while (stack.length > 0) {
        const [cx, cy] = stack.pop();
        const currentColor = this.getPixel(data, cx, cy);

        // currentColor가 targetColor와 tolerance 범위 내에서 같아야 채우기
        if (!this.colorsMatch(currentColor, targetColor, this.tolerance)) {
          continue;
        }

        // 해당 위치 색상 변경
        this.setPixel(data, cx, cy, fillColor);

        // 8방향을 확인해서 스택에 추가
        for (const [dx, dy] of directions) {
          const nx = cx + dx;
          const ny = cy + dy;
          // 화면 범위 벗어나지 않는지 체크
          if (nx >= 0 && nx < width && ny >= 0 && ny < height) {
            stack.push([nx, ny]);
          }
        }
      }
    },
    // (x, y) 픽셀의 RGBA 정보 반환
    getPixel(data, x, y) {
      const index = (y * this.canvas.width + x) * 4;
      return [
        data[index],
        data[index + 1],
        data[index + 2],
        data[index + 3]
      ];
    },
    // (x, y) 픽셀에 color(r, g, b, a) 설정
    setPixel(data, x, y, color) {
      const index = (y * this.canvas.width + x) * 4;
      data[index] = color[0];
      data[index + 1] = color[1];
      data[index + 2] = color[2];
      data[index + 3] = color[3] !== undefined ? color[3] : 255; // alpha 값 없으면 255(불투명)
    },
    // 색상 비교(오차 범위 tolerance 사용)
    colorsMatch(color1, color2, tolerance = 0) {
      return (
        Math.abs(color1[0] - color2[0]) <= tolerance &&
        Math.abs(color1[1] - color2[1]) <= tolerance &&
        Math.abs(color1[2] - color2[2]) <= tolerance
      );
    },
    // 문자열 색 -> RGB 배열 변환
    hexToRgb(color) {
      // "rgb(255, 0, 0)" 형태인지 체크
      if (color.startsWith("rgb")) {
        return color.match(/\d+/g).map(Number);
      }

      // 미리 정의된 색상 이름
      const predefinedColors = {
        red: [255, 0, 0],
        blue: [0, 0, 255],
        green: [0, 255, 0],
        yellow: [255, 255, 0],
        pink: [255, 192, 203],
        orange: [255, 165, 0]
      };

      if (color in predefinedColors) {
        return predefinedColors[color];
      }

      // HEX(#RRGGBB) 형태
      const bigint = parseInt(color.replace("#", ""), 16);
      return [
        (bigint >> 16) & 255,
        (bigint >> 8) & 255,
        bigint & 255
      ];
    },
    // 원본 이미지로 리셋
    resetCanvas() {
      if (this.originalImageData) {
        this.ctx.putImageData(this.originalImageData, 0, 0);
      }
    },
    // 사용자 커스텀 색상 추가
    addCustomColor() {
      const { r, g, b } = this.customColor;
      const rgbString = `rgb(${r}, ${g}, ${b})`;
      if (!this.colors.includes(rgbString)) {
        this.colors.push(rgbString);
      }
    }
  },
  // 컴포넌트가 마운트되면 캔버스 초기화
  mounted() {
    this.initCanvas();
  }
};
</script>

<style scoped>
.avatar-canvas {
  border: 1px solid #ccc;
  display: block;
  margin-bottom: 20px;
}

.color-picker {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

.color-box {
  width: 30px;
  height: 30px;
  cursor: pointer;
  border: 1px solid #000;
  transition: transform 0.2s ease;
}

.color-box:hover {
  transform: scale(1.2);
}

.eraser {
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 16px;
  background-color: #fff;
  color: black;
}

.custom-color {
  margin: 20px 0;
  display: flex;
  align-items: center;
  gap: 10px;
}

.custom-color input {
  width: 50px;
  padding: 5px;
  border: 1px solid #ccc;
  border-radius: 5px;
}

.color-preview {
  width: 30px;
  height: 30px;
  border: 1px solid #000;
}

label {
  display: inline-block;
  margin-right: 1rem;
}
</style>
