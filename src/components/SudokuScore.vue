<template>
  <div class="text-center">
    {{ score }}
  </div>
</template>

<script setup lang="ts">
import { ref, watch } from 'vue';


const score = ref(0)
const answerPoints = 420

const props = defineProps<{
    seconds: number,
    answers: number,
    mistakes: number,
}>()

watch(
  () => props.answers,
  (newVal, oldVal) => {
    if (newVal !== oldVal) {
        if (newVal === 0) {
            score.value = 0
            return
        }
        addScore()
    }
  }
)

watch(
  () => props.mistakes,
  (newVal, oldVal) => {
    if (newVal !== oldVal) {
      addMistake()
    }
  }
)

const addScore = () => {
    const lessPoints = Math.floor(props.seconds/10)
    const pointsToWin = answerPoints - lessPoints
    score.value = score.value + pointsToWin
}

const mistakePoints = 700

const addMistake = () => {
    const pointsToRemove = mistakePoints * props.mistakes
    if (score.value - pointsToRemove <= 0){
        score.value = 0
        return
    }
    score.value = score.value - pointsToRemove
}





</script>

<style scoped>
</style>
