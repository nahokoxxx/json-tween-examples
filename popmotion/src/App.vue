<script setup lang="ts">
import { onMounted, ref, watch } from "vue";
import dataList from "../pickle_5.json";
import { animate } from "popmotion";

type Index = keyof typeof dataList;
type Car = {
  car_type: number;
  car_pos: number[];
  car_angle: number;
  car_length: number;
};

const initialIndex = Object.keys(dataList)[0] as Index;

const canvasRef = ref<HTMLCanvasElement | null>(null);
const currentIndex = ref(initialIndex);
const playingIntervalId = ref<number | null>(null);

let drawnCars: { [id: string]: Car } = dataList[initialIndex];

watch(currentIndex, (value, oldValue) => {
  const data: { [id: string]: Car } = { ...dataList[value] };
  const previousData: { [id: string]: Car } = { ...dataList[oldValue] };

  const carIds = Object.keys(data);
  // carIdsにインデックスが含まれない場合は描画対象から除外する
  drawnCars = Object.keys(drawnCars).reduce((acc, id) => {
    if (carIds.includes(id)) {
      acc[id] = drawnCars[id];
    }
    return acc;
  }, {} as { [id: string]: Car });

  carIds.forEach((id) => {
    const carData = data[id];
    const previousCarData = previousData[id];

    if (previousCarData) {
      animate({
        from: { x: previousCarData.car_pos[0], y: previousCarData.car_pos[1] },
        to: { x: carData.car_pos[0], y: carData.car_pos[1] },
        duration: 1000,
        onUpdate: (updatedData: { x: number; y: number }) => {
          drawnCars[id] = {
            ...carData,
            car_pos: [updatedData.x, updatedData.y],
          };
        },
      });
    } else {
      drawnCars[id] = carData;
    }
  });
});

onMounted(() => {
  requestAnimationFrame(animateDraw);
});

function animateDraw() {
  draw();
  requestAnimationFrame(animateDraw);
}

function getCanvas2dContext() {
  if (canvasRef.value === null) {
    throw new Error("canvas is not mounted");
  }
  return canvasRef.value.getContext("2d") as CanvasRenderingContext2D;
}

function draw() {
  const ctx = getCanvas2dContext();
  ctx.clearRect(0, 0, 460, 480);

  Object.values(drawnCars).forEach((car) => {
    ctx.fillStyle = "blue";
    ctx.fillRect(car.car_pos[0], car.car_pos[1], 10, 10);
  });
}

function play() {
  playingIntervalId.value = setInterval(() => {
    const numberCurrentIndex = Number(currentIndex.value);
    const nextIndex = (numberCurrentIndex + 1).toString() as Index;
    currentIndex.value = nextIndex;
    if (
      nextIndex === Object.keys(dataList)[Object.keys(dataList).length] &&
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

function seek(index: Index) {
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
