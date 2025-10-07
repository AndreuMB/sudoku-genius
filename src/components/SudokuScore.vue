<template>
  <div class="text-center">
    {{ score }}
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref, watch } from 'vue';


const score = ref(0)
const mistakes = ref(0)
const answerPoints = 420

const props = defineProps<{
    seconds: number
}>()


const addScore = () => {
  const lessPoints = Math.floor(props.seconds/10)
  const pointsToWin = answerPoints - lessPoints
  score.value = score.value + pointsToWin
  localStorage.setItem('score', score.value + '')
}

const mistakePoints = 700

const addMistake = () => {
  mistakes.value++
  const pointsToRemove = mistakePoints * mistakes.value
  if (score.value - pointsToRemove <= 0){
      score.value = 0
      return
  }
  score.value = score.value - pointsToRemove
  localStorage.setItem('score', score.value + '')
  localStorage.setItem('mistakes', mistakes.value+'')
}

defineExpose({
  addScore,
  addMistake
})

onMounted(()=> {
  score.value = Number(localStorage.getItem('score')) | 0
  mistakes.value = Number(localStorage.getItem('mistakes')) | 0
})





</script>

<style scoped>
</style>
