<script setup lang="ts">
import { onMounted, ref, watch } from "vue";
import dataList from "../data.json";
import gsap from "gsap";

const canvasRef = ref<HTMLCanvasElement | null>(null);
const currentIndex = ref(0);
const playingIntervalId = ref<number | null>(null);

let currentData = { ...dataList[0] };

watch(currentIndex, (value) => {
  const data = { ...dataList[value] };

  gsap.to(currentData, {
    ...data,
    duration: 1,
    onUpdate: () => {
      drawRect();
    },
  });
});

onMounted(() => {
  drawRect();
});

function getCanvas2dContext() {
  if (canvasRef.value === null) {
    throw new Error("canvas is not mounted");
  }
  return canvasRef.value.getContext("2d") as CanvasRenderingContext2D;
}

function drawRect() {
  const ctx = getCanvas2dContext();
  ctx.clearRect(0, 0, 460, 480);
  ctx.fillStyle = "red";
  ctx.fillRect(currentData.x, currentData.y, 100, 100);
}

function play() {
  playingIntervalId.value = setInterval(() => {
    currentIndex.value++;
    if (
      currentIndex.value >= dataList.length - 1 &&
      playingIntervalId.value !== null
    ) {
      clearInterval(playingIntervalId.value);
    }
  }, 1000);
}

function stop() {
  if (playingIntervalId.value !== null) {
    clearInterval(playingIntervalId.value);
    playingIntervalId.value = null;
  }
}

function seek(index: number) {
  stop();
  currentIndex.value = index;
}
</script>

<template>
  <div>currentIndex: {{ currentIndex }}</div>
  <canvas ref="canvasRef" width="460" height="480" class="canvas" />
  <div>
    <button v-if="playingIntervalId" class="button" @click="stop">停止</button>
    <button v-else class="button" @click="play">再生</button>
    <div>
      シーク
      <button
        v-for="(_, index) in dataList"
        :key="index"
        class="button"
        @click="seek(index)"
      >
        {{ index }}
      </button>
    </div>
  </div>
</template>

<style scoped>
.canvas {
  background-color: black;
}

.button {
  background-color: lightgray;
}
</style>
