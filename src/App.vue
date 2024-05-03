<script setup>
  import { onMounted, ref } from 'vue'

  var canvasWidth = ref(100);
  var canvasHeight = ref(100);

  var probability = ref(10);
  const speed = 300;
  const cellColor = "green"

  var canvasContext = null;
  var nextFrameContext = null;

  var iteration;

  onMounted(() => {
    var canvas = document.getElementById("canvas");
    canvasContext = canvas.getContext("2d", { alpha: false, willReadFrequently: true });

    var nextFrame = document.getElementById("nextFrame");
    nextFrameContext = nextFrame.getContext("2d", { alpha: false, willReadFrequently: true });
  })

  function startState() {
    // Randomly fills the whole canvas with living cells
    for (let y = 0; y < canvasHeight.value; y++) {
      for (let x = 0; x < canvasWidth.value; x++) {
        let rnd = Math.random() <= probability.value / 100;
        if (rnd) {
          canvasContext.fillStyle = "red";
          canvasContext.fillRect(x, y, 1, 1);
        } else {
          canvasContext.fillStyle = "black";
          canvasContext.fillRect(x, y, 1, 1);
        }
      }
    }
  }

  function step() {
    const imageData = canvasContext.getImageData(0, 0, canvasWidth.value, canvasHeight.value);
    for (let y = 0; y < canvasHeight.value; y++) {
      for (let x = 0; x < canvasWidth.value; x++) {
        let neighbourCellCount = 0;
        const isAlive = imageData.data[getPixelIndex(x, y)] === 255;
        
        // 1 2 3
        // 4 x 5
        // 6 7 8
        const pixelsToCheck = [];
        pixelsToCheck.push([x - 1, y - 1]); // 1
        pixelsToCheck.push([x, y - 1]); // 2
        pixelsToCheck.push([x + 1, y - 1]); // 3
        pixelsToCheck.push([x - 1, y]); // 4
        pixelsToCheck.push([x + 1, y]); // 5
        pixelsToCheck.push([x - 1, y + 1]); // 6
        pixelsToCheck.push([x, y + 1]); // 7
        pixelsToCheck.push([x + 1, y + 1]); // 8

        pixelsToCheck.forEach((element) => {
          if (element[0] < 0) {
            element[0] = canvasWidth.value - element[0]
          } else if (element[0] > canvasWidth.value) {
            element[0] = element[0] - canvasWidth.value;
          }

          if (element[1] < 0) {
            element[1] = canvasHeight.value - element[1]
          } else if (element[1] > canvasHeight.value) {
            element[1] = element[1] - canvasHeight.value;
          }

          const pixel = imageData.data[getPixelIndex(element[0], element[1])];

          if (pixel === 255) {
            neighbourCellCount++;
          }
        })

        if (neighbourCellCount === 3 && !isAlive) {
          // Any dead cell with exactly three live neighbors becomes a live cell
          nextFrameContext.fillStyle = "red";
          nextFrameContext.fillRect(x, y, 1, 1);
        } else if (neighbourCellCount < 2 && isAlive) {
          // Any live cell with fewer than two live neighbors dies
          nextFrameContext.fillStyle = "black";
          nextFrameContext.fillRect(x, y, 1, 1);
        } else if ((neighbourCellCount === 2 || neighbourCellCount === 3) && isAlive) {
          // Any live cell with two or three live neighbors lives on to the next generation
          nextFrameContext.fillStyle = "red";
          nextFrameContext.fillRect(x, y, 1, 1);
        } else if (neighbourCellCount > 3 && isAlive) {
          // Any live cell with more than three live neighbors dies
          nextFrameContext.fillStyle = "black";
          nextFrameContext.fillRect(x, y, 1, 1);
        } else {
          nextFrameContext.fillStyle = "black";
          nextFrameContext.fillRect(x, y, 1, 1);
        }
      }
    }

    canvasContext.drawImage(nextFrame, 0, 0);
  }

  function start() {
    if (!iteration) {
      iteration = setInterval(step, speed);
    }
  }

  function stop() {
    clearInterval(iteration);
    iteration = null;
  }

  // Calculates the index of a pixel in a canvas image data array
  function getPixelIndex(x, y) {
    return y * (canvasWidth.value * 4) + x * 4
  }

  function onInput() {
      // this.currentValue is a string because HTML is weird
      this.$emit('input', parseInt(probability.value));
  }
</script>

<template>
  <header></header>
  <main>
    <div class="container">
      <h1>Conway's Game of Life in Vue.js</h1>
      <div class="flex-container">
        <canvas id="canvas" :width="canvasWidth" :height="canvasHeight"></canvas>
        <canvas id="nextFrame" :width="canvasWidth" :height="canvasHeight"></canvas>
      </div>
      <div class="flex-container" id="controls">
        <button class="button" @click="startState">Generate</button>
        <div class="slider-container">
          <div><strong>Initial living Cells</strong></div>
          <input type="range" min="10" max="90" class="slider" v-model="probability" @input="onInput">
          <span><strong>{{ probability }}%</strong></span>
        </div>
        <button class="button" @click="start">Start</button>
        <button class="button" @click="step">Step</button>
        <button class="button" @click="stop">Stop</button>
      </div>
    </div>
  </main>
</template>

<style scoped>
  main {
    font-family: sans-serif
  }

  canvas {
    border: 1px solid black;
  }

  .container {
    width: fit-content;
    margin-inline-start: auto;
    margin-inline-end: auto;
    padding: 24px;
  }

  .flex-container {
    display: flex;
    gap: 10px;
    margin-block-end: 10px;
    justify-content: space-around;
  }

  #nextFrame {
    display: none;
  }

  #canvas {
    width: 600px;
    height: 600px;
  }

  .button {
    padding: 8px 16px;

    background-color: #6200EE;
    border-radius: 4px;
    border: 1px solid #6200EE;
    box-shadow: rgba(0, 0, 0, 0.15) 1.95px 1.95px 2.6px;
    color: white;
    font-weight: bold;
    text-transform: uppercase;
  }

  .button:hover {
    background-color: #803ff2;
    border: 1px solid #803ff2;
    box-shadow: rgba(0, 0, 0, 0.25) 1.95px 1.95px 2.6px;
  }
</style>