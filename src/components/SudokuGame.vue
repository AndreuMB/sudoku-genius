<template>
  <div id="container" class="flex flex-col h-full w-full p-6 justify-center text-center items-center">
    <div class="flex flex-col gap-6">
        <div class="w-full">
            <div class="w-full flex justify-between *:w-30">
                <SudokuTimer class="text-left" ref="timerRef" @seconds="(sec) => handleSenconds(sec)" />
                <SudokuScore :seconds :answers="answers" :mistakes="mistakes" />
                <p class="text-right">Mistakes: {{ mistakes }}/3</p>
            </div>
        <div class="w-full max-w-xl aspect-square">
            <table class="bg-primary w-full h-full table-fixed text-lg">
                <tr class="sudoku-number" v-for="(line) in sudokuLines">
                    <td 
                        v-for="numberPosition in line"
                        @click="(ev) => selectCell(ev,numberPosition)" 
                        class="w-5 h-5 relative" 
                        :class="[numberPosition.number === null ? 'bg-primary-dark' : '', numberPosition.color, selectedPosition === numberPosition ? 'bg-primary!' : '']"
                    >
                        <div
                            v-show="numberPosition.number === null"
                            class="note-number absolute h-full w-full top-0 left-0 grid grid-cols-3 grid-rows-3 text-sm text-gray-400"
                        >
                            <div class="1"></div>
                            <div class="2"></div>
                            <div class="3"></div>
                            <div class="4"></div>
                            <div class="5"></div>
                            <div class="6"></div>
                            <div class="7"></div>
                            <div class="8"></div>
                            <div class="9"></div>
                        </div>
                        <div v-if="numberPosition.number !== null" >
                            {{ numberPosition.number + 1 }}
                        </div>
                    </td>
                </tr>
            </table>
        </div>
        </div>
        <div class="flex justify-center w-full buttons items-center flex-wrap gap-2">
            <button @click="addNumber(value)" class="" v-for="value in [1,2,3,4,5,6,7,8,9]">{{ value }}</button>
            <button 
                @click="notesToggle=!notesToggle"
                :class="[notesToggle ? 'bg-secondary' : '']"
            >
                <ion-icon name="create-outline"></ion-icon>
            </button>
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
import { IonAlert, IonIcon } from '@ionic/vue';
import { makepuzzle, solvepuzzle, ratepuzzle } from "sudoku";
import { ref } from "vue";
import SudokuTimer from './SudokuTimer.vue';
import SudokuScore from './SudokuScore.vue';


const alertButtons = ['Try again'];
const isOpen = ref(false);
const sudoku = ref<Array<any>>(makepuzzle())
const sudokuSolved = ref<Array<any>>(solvepuzzle(sudoku))
const answers = ref(0)
const mistakes = ref(0)

const timerRef = ref<InstanceType<typeof SudokuTimer> | null>(null)
const seconds = ref(0)

const handleSenconds = (secondsTimer:number) => {   
    seconds.value = secondsTimer
}

const notesToggle = ref(false)


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
    answers.value = 0
    if (timerRef.value) timerRef.value.reset()
    
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

generateSudoku()

const closeAlert = () => {
    isOpen.value = false
    generateSudoku()
};

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

    // sudoku numbers internal function go from 0 to 8
    number = number - 1

    const correctNumber = sudokuSolved.value[selectedPosition.value.index]
    
    if (selectedPosition.value.number === correctNumber) return

    if (notesToggle.value) {
        selectedPosition.value.number = null
        notesEnabled(selectedElement.value,number)
        return
    }

    if (number === correctNumber) {
        // correct
        selectedPosition.value.color = 'text-secondary!'
        answers.value++
    } else {
        // mistake
        if (mistakes.value>=3) {
            isOpen.value = true
            if (timerRef.value) timerRef.value.stop()
            return
        }
        mistakes.value++
        selectedPosition.value.color = 'text-red-400!'
    }    
    selectedPosition.value.number = number
}

const notesEnabled = (cell:HTMLElement, number:number) => {
    const fixNumber = number + 1 + ''
    const cellGridNumber = cell.getElementsByClassName(fixNumber)[0]
    if (!cellGridNumber) return
    console.log('content', cellGridNumber.textContent);
    
    cellGridNumber.textContent = !cellGridNumber.textContent ? fixNumber : ''
}


</script>

<style scoped>
    td {
        border:2px solid var(--color-secondary);
    }
    table {
        border:3px solid  var(--color-secondary);

    }
    tr:nth-child(3n){
        border-bottom:3px solid var(--color-secondary);
    }

    td:nth-child(3n) {
        border-right:3px solid var(--color-secondary);
    }

    @media (max-width: 500px) {
        .sudoku-number{
            font-size: 12px;
        }
        .note-number {
            font-size: 9px;
        }
    }

    @media (max-width: 350px) {
        .sudoku-number{
            font-size: 10px;
        }
        .note-number {
            font-size: 7px;
        }
    }

    .buttons > button {
        border: solid 1px var(--color-primary);
        border-radius: 25%;
        width: 2.5em;
        height: 2.5em;
        font-size: larger;
    }

    .buttons > button:hover{
        background-color: var(--color-primary);
    }

    .buttons > button:active{
        background-color: var(--color-primary-dark);
    }
</style>