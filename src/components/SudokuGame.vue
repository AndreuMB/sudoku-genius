<template>
  <div id="container" class="flex flex-col h-full w-full p-6 justify-center text-center items-center">
    <div class="flex flex-col gap-6">
        <div class="w-full">
            <div class="w-full flex justify-between *:w-full">
                <SudokuTimer class="text-left" ref="timerRef" @seconds="(sec) => handleSenconds(sec)" />
                <SudokuScore :seconds :answers="answers" :mistakes="mistakes" />
                <p class="text-right">Mistakes: {{ mistakes }}/3</p>
            </div>
            <div class="w-full max-w-xl aspect-square">
                <table class="w-full h-full table-fixed text-2xl">
                    <tr class="sudoku-number bg-primary" v-for="(line, lineIndex) in sudoku">
                        <td 
                            v-for="(numberPosition, colIndex) in line"
                            @click="(ev) => selectCell(ev,numberPosition)" 
                            class="w-5 h-5 relative" 
                            :class="[
                                numberPosition.number === null ? '' : '',
                                numberPosition.color,
                                selectedRow === lineIndex ? 'bg-terciary-light' : '',
                                selectedCol === colIndex ? 'bg-terciary-light' : '',
                                selectedPosition === numberPosition ? 'bg-terciary!' : '',
                                selectedSquare && 
                                Math.floor(lineIndex / 3) === selectedSquare.row &&
                                Math.floor(colIndex / 3) === selectedSquare.col 
                                    ? 'bg-terciary-light' 
                                    : ''
                                
                            ]"
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
        <div class="flex flex-col gap-2">
            <div class="buttons">
                <!-- first row buttons -->
            </div>
            <div class="flex justify-center w-full buttons items-center flex-wrap gap-2">
                <button @click="addNumber(value)" v-for="value in [1,2,3,4,5,6,7,8,9]">{{ value }}</button>
                <button 
                    @click="notesToggle=!notesToggle"
                    :class="[notesToggle ? 'bg-primary-dark' : 'bg-secondary']"
                >
                    <ion-icon name="create-outline"></ion-icon>
                </button>
            </div>
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
import { computed, onMounted, ref } from "vue";
import SudokuTimer from './SudokuTimer.vue';
import SudokuScore from './SudokuScore.vue';

interface NumberPosition {
    number: number | null, // display number
    index: number, // check in sudokuSolved
    color: string // color number
}

const alertButtons = ['Try again'];
const isOpen = ref(false);
const sudokuGen = ref<Array<any>>(makepuzzle())
const sudokuSolved = ref<Array<any>>(solvepuzzle(sudokuGen))
const sudokuUserSolved = ref<Array<number|null>>([])
const answers = ref(0)
const mistakes = ref(0)

const timerRef = ref<InstanceType<typeof SudokuTimer> | null>(null)
const seconds = ref(0)
const notesToggle = ref(false)

const sudoku = ref<Array<Array<NumberPosition>>>([])
const selectedElement = ref<HTMLElement | null>(null)
const selectedPosition = ref<NumberPosition | null>(null)

const answersColor = 'text-secondary-dark'
const mistakesColor = 'text-extra'

const selectedRow = computed(() => {
  if (!selectedPosition.value) return null  
  return Math.floor(selectedPosition.value.index / 9)
})

const selectedCol = computed(() => {
  if (!selectedPosition.value) return null 
  return selectedPosition.value.index % 9
})

const selectedSquare = computed(() => {
if (selectedRow.value === null || selectedCol.value === null) return null
  return {
    row: Math.floor(selectedRow.value / 3),
    col: Math.floor(selectedCol.value / 3),
  }
})

onMounted(()=> {
    const sudokuLS = localStorage.getItem('sudoku')
    const sudokuUserSolvedLS = localStorage.getItem('sudokuUserSolved')

    if (sudokuLS) {
        let sudokuArray = sudokuStringtoArray(sudokuLS)
        if (sudokuUserSolvedLS) {
            sudokuUserSolved.value = sudokuStringtoArray(sudokuUserSolvedLS)
            // sudokuArray = addUserSolved(sudokuArray)
            generateSudoku(sudokuArray, sudokuUserSolved.value)
            return
        } 
        generateSudoku(sudokuArray)
        return
    }

    generateSudoku(makepuzzle())

    
})

const sudokuStringtoArray = (sudoku:string): Array<number|null> => {
    const sudokuArray = sudoku.split(',')        
    const sudokuNum = sudokuArray.map((str)=> str === '' ? null : Number(str))
    return sudokuNum
}

const handleSenconds = (secondsTimer:number) => {   
    seconds.value = secondsTimer
}

const generateSudoku = (sudokuG: Array<number | null>, sudokuUserSoluved?: Array<number | null>) => {
    sudokuGen.value = sudokuG
    localStorage.setItem('sudoku',sudokuGen.value.toString())   
    sudokuSolved.value = solvepuzzle(sudokuGen.value)
    mistakes.value = Number(localStorage.getItem('mistakes')) | 0
    answers.value = Number(localStorage.getItem('answers')) | 0

    if (timerRef.value) timerRef.value.reset()
    
    sudoku.value = []

    if (selectedElement.value) selectedElement.value.classList.remove('bg-gray-700!')
    selectedElement.value = null

    selectedPosition.value = null

    let indexSudoku = 0
    // 9 lines
    for (let line = 0; line < 9; line++) {
        const newLine = []
        // 9 numbers
        for (let index = 0; index < 9; index++) {
            const number = sudokuGen.value[indexSudoku]
            let newNumber = {number:number, index: indexSudoku, color:''}

            if (sudokuUserSoluved && Number.isInteger(sudokuUserSoluved[indexSudoku])) {
                const userSolvedNum = sudokuUserSoluved[indexSudoku]
                newNumber = {number:userSolvedNum, index: indexSudoku, color: answersColor}
            }

            newLine.push(newNumber)
            indexSudoku++ 
        }
        sudoku.value.push(newLine)
    }
}

const closeAlert = () => {
    isOpen.value = false
    generateSudoku(makepuzzle())
};

const selectCell = (ev:PointerEvent, numberPosition: NumberPosition) => {
    const td = ev.currentTarget as HTMLElement
    // td.classList.add('bg-gray-700')
    // if (selectedElement.value) selectedElement.value.classList.remove('bg-gray-700!')
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
        selectedPosition.value.color = answersColor
        answers.value++
        localStorage.setItem(answers.value+'','answers')
        // sudoku.value[selectedPosition.value.index] = number
        // localStorage.setItem('sudoku',sudoku.value.toString())
        sudokuUserSolved.value[selectedPosition.value.index] = number
        localStorage.setItem('sudokuUserSolved',sudokuUserSolved.value.toString())
    } else {
        // mistake
        if (mistakes.value>=3) {
            isOpen.value = true
            if (timerRef.value) timerRef.value.stop()
            return
        }
        mistakes.value++
        localStorage.setItem(mistakes.value+'','mistakes')
        selectedPosition.value.color = mistakesColor
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
        color: var(--terciary-dark-color);
    }
    tr:nth-child(3n){
        border-bottom:3px solid var(--color-secondary);
    }

    td:nth-child(3n) {
        border-right:3px solid var(--color-secondary);
    }

    @media (max-width: 500px) {
        .sudoku-number{
            font-size: 20px;
        }
        .note-number {
            font-size: 9px;
        }
    }

    @media (max-width: 350px) {
        .sudoku-number{
            font-size: 14px;
        }
        .note-number {
            font-size: 7px;
        }
    }

    .buttons > button {
        border: solid 1px var(--color-primary);
        border-radius: 25%;
        max-width: 2.5em;
        max-height: 2.5em;
        min-width: 2em;
        min-height: 2em;
        width: 2.5em;
        height: 2.5em;
        font-size: larger;
    }

    .buttons > button:hover{
        background-color: var(--color-primary);
    }

    /* .buttons > button:active{
        background-color: var(--color-primary-dark);
    } */
</style>