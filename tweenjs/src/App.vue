<script setup lang="ts">
// @ts-expect-error
import * as TWEEN from "@tweenjs/tween.js";
import { onMounted, ref, watch } from "vue";
import dataList from "../data.json";

const canvasRef = ref<HTMLCanvasElement | null>(null);
const currentIndex = ref(0);
const playingIntervalId = ref<number | null>(null);

watch(currentIndex, (value, oldValue) => {
  const data = { ...dataList[value] };
  const previousData = { ...dataList[oldValue] };

  const tween = new TWEEN.Tween(previousData)
    .to(data, 1000)
    .onUpdate((updatedData: { x: number; y: number }) => {
      drawRect(updatedData);
    })
    .start();
  requestAnimationFrame(animate);

  function animate(time: number) {
    tween.update(time);
    requestAnimationFrame(animate);
  }
});

onMounted(() => {
  drawRect(dataList[0]);
});

function getCanvas2dContext() {
  if (canvasRef.value === null) {
    throw new Error("canvas is not mounted");
  }
  return canvasRef.value.getContext("2d") as CanvasRenderingContext2D;
}

function drawRect(data: { x: number; y: number }) {
  const ctx = getCanvas2dContext();
  ctx.clearRect(0, 0, 460, 480);
  ctx.fillStyle = "red";
  ctx.fillRect(data.x, data.y, 100, 100);
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
