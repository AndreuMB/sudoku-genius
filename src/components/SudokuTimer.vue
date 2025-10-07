<template>
  <div>
    {{ timerString }}
  </div>
</template>

<script setup lang="ts">
import { onMounted, onUnmounted, ref } from 'vue';

const emits = defineEmits(['seconds'])
const timerString = ref('00:00')
const seconds = ref(0)

let timer: ReturnType<typeof setInterval> | null = null

const start = (savedSeconds:number) => {
  console.log('savedSeconds',savedSeconds);
  seconds.value = savedSeconds
  if (timer) return // avoid multiple intervals
  timer = setInterval(() => {
    emits('seconds', seconds.value)
    timerString.value = convertSecondstoTime(seconds.value)
    seconds.value++
  }, 1000);
}

const stop = () => {
  if (timer) {
    clearInterval(timer)
    timer = null
  }
}

const reset = () => {
  stop()
  seconds.value = 0
  timerString.value = '00:00'
  start(seconds.value)
}

// expose methods to parent
defineExpose({
  reset,
  start,
  stop,
})

onUnmounted(() => {
  stop()
})

function convertSecondstoTime(totalSeconds: number) {
  const dateObj = new Date(totalSeconds * 1000);
  const minutes = dateObj.getUTCMinutes();
  const seconds = dateObj.getSeconds();

  return (
    minutes.toString().padStart(2, '0') +
    ':' +
    seconds.toString().padStart(2, '0')
  )
}
</script>
