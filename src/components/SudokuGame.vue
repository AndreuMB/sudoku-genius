<template>
  <div id="container" class="flex flex-col h-full w-full p-6 gap-12 justify-center text-center items-center">
    <div class="flex flex-col gap-6 w-full max-w-90">
        <div class="w-full text-right"><p>Mistakes: {{ mistakes }}/3</p></div>
        <table class="bg-gray-900 w-full">
            <tr class="sudoku-number" v-for="(line) in sudokuLines">
                <td 
                    v-for="numberPosition in line"
                    @click="(ev) => selectCell(ev,numberPosition)" 
                    class="w-4! h-10!" 
                    :class="[numberPosition.number === null ? 'bg-gray-800' : '', numberPosition.color, selectedPosition === numberPosition ? 'bg-gray-700!' : '']"
                >                
                    {{ numberPosition.number === null ? ' ' : numberPosition.number + 1 }}
                </td>
            </tr>
        </table>
        <div class="flex justify-between w-full">
            <button @click="addNumber(value)" class="border! p-2! rounded! hover:bg-gray-700! active:bg-gray-700!" v-for="value in [1,2,3,4,5,6,7,8,9]">{{ value }}</button>
        </div>
    </div>
    <ion-alert
        header="Fail"
        message="Uh Oh... You made more than 3 mistakes."
        :is-open="isOpen"
        :buttons="alertButtons"
        @didDismiss="closeAlert()"
    >

    </ion-alert>
  </div>


</template>

<script setup lang="ts">
import { IonAlert, IonButton } from '@ionic/vue';
import { makepuzzle, solvepuzzle, ratepuzzle } from "sudoku";
import { ref } from "vue";

const alertButtons = ['Try again'];
const isOpen = ref(false);
const sudoku = ref<Array<any>>(makepuzzle())
const sudokuSolved = ref<Array<any>>(solvepuzzle(sudoku))
const mistakes = ref(0)


interface NumberPosition {
    number: number | null,
    index: number,
    color: string
}

const sudokuLines = ref<Array<Array<NumberPosition>>>([])
const selectedElement = ref<HTMLElement | null>(null)
const selectedPosition = ref<NumberPosition | null>(null)

const generateSudoku = () => {
    sudoku.value = makepuzzle()
    sudokuSolved.value = solvepuzzle(sudoku.value)
    mistakes.value = 0
    sudokuLines.value = []

    if (selectedElement.value) selectedElement.value.classList.remove('bg-gray-700!')
    selectedElement.value = null

    selectedPosition.value = null

    let indexSudoku = 0
    // 9 lines
    for (let line = 0; line < 9; line++) {
        const newLine = []
        // 9 numbers
        for (let index = 0; index < 9; index++) {
            const number = sudoku.value[indexSudoku]
            newLine.push({number:number, index: indexSudoku, color:''})
            indexSudoku++ 
        }
        sudokuLines.value.push(newLine)
    }
}

const closeAlert = () => {
    isOpen.value = false
    generateSudoku()
};

generateSudoku()

const selectCell = (ev:PointerEvent, numberPosition: NumberPosition) => {
    const td = ev.currentTarget as HTMLElement
    td.classList.add('bg-gray-700!')
    if (selectedElement.value) selectedElement.value.classList.remove('bg-gray-700!')
    selectedElement.value = td
    selectedPosition.value = numberPosition
}

const addNumber = (number: number) => {
    if (!selectedPosition.value) return
    if (!selectedElement.value) return

    const correctNumber = sudokuSolved.value[selectedPosition.value.index] + 1

    console.log('selectedNum',selectedPosition.value.number);
    console.log('correctNum',correctNumber);
    
    if (selectedPosition.value.number === correctNumber) return

    if (number === correctNumber) {
        // correct
        selectedPosition.value.color = 'text-gray-400!'
    } else {
        // mistake
        if (mistakes.value>=3) {
            isOpen.value = true
            return
        }
        mistakes.value++
        selectedPosition.value.color = 'text-red-400!'
    }    
    selectedPosition.value.number = number-1

}


</script>

<style scoped>
    td {
        border:1px solid white;
    }
    table {
        border:3px solid white;

    }

    tr:nth-child(3n){
        border-bottom:3px solid white;
    }

    td:nth-child(3n) {
        border-right:3px solid white;
    }
</style>