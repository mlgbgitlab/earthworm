<template>
  <div class="flex items-center font-sans text-gray-300 dark:text-gray-500">
    <div
      ref="clockIcon"
      class="mr-1 flex items-center justify-center"
    >
      <span
        class="i-ph-alarm-bold"
        style="width: 30px; height: 30px"
      ></span>
    </div>
    <p class="text-lg font-bold">{{ formattedTime }}</p>
  </div>
</template>

<script setup>
import { computed, onMounted, onUnmounted, ref, watch } from "vue";

import { useLearningTimeTracker } from "~/composables/main/learningTimeTracker";

const { $anime } = useNuxtApp();

const { totalSeconds, startTracking, stopTracking } = useLearningTimeTracker();
const clockIcon = ref(null);

const formattedTime = computed(() => {
  const hours = Math.floor(totalSeconds.value / 3600);
  const minutes = Math.floor((totalSeconds.value % 3600) / 60);
  const seconds = totalSeconds.value % 60;
  return `${hours.toString().padStart(2, "0")}:${minutes.toString().padStart(2, "0")}:${seconds.toString().padStart(2, "0")}`;
});

function animateClock() {
  $anime({
    targets: clockIcon.value,
    translateY: [
      { value: -4, duration: 100, easing: "easeInOutQuad" },
      { value: 4, duration: 200, easing: "easeInOutQuad" },
      { value: -4, duration: 200, easing: "easeInOutQuad" },
      { value: 4, duration: 200, easing: "easeInOutQuad" },
      { value: 0, duration: 100, easing: "easeInOutQuad" },
    ],
    rotate: [
      { value: -5, duration: 100, easing: "easeInOutQuad" },
      { value: 5, duration: 200, easing: "easeInOutQuad" },
      { value: -5, duration: 200, easing: "easeInOutQuad" },
      { value: 5, duration: 200, easing: "easeInOutQuad" },
      { value: 0, duration: 100, easing: "easeInOutQuad" },
    ],
    scale: [
      { value: 1.1, duration: 400, easing: "easeInOutQuad" },
      { value: 1, duration: 400, easing: "easeInOutQuad" },
    ],
    duration: 800,
    loop: 1,
  });
}

watch(totalSeconds, (newValue) => {
  if (newValue % 60 === 0 && newValue !== 0) {
    animateClock();
  }
});

// 因为这个组件是放在 game.vue 中的
// 所以生命周期是跟着 game 一起的
// 所以我们在这里启动 tracking 等于同游戏开始的时候启动  游戏结束的时候停止
// 放在这里的原因是在 game.vue 里面控制了 learningTimer 的渲染
// 所以对于 startTracking 就不需要在做额外的判断了
onMounted(() => {
  startTracking();
});

onUnmounted(() => {
  stopTracking();
});

function handleVisibilityChange() {
  if (document.hidden) {
    stopTracking();
  } else {
    startTracking();
  }
}

onMounted(() => {
  document.addEventListener("visibilitychange", handleVisibilityChange);
  window.addEventListener("beforeunload", stopTracking);
});

onUnmounted(() => {
  document.removeEventListener("visibilitychange", handleVisibilityChange);
  window.removeEventListener("beforeunload", stopTracking);
});
</script>
