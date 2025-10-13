<template>
  <div id="container" class="flex flex-col h-full w-full p-2 justify-center text-center items-center">
    <div class="flex flex-col gap-6">
        <div class="w-full">
            <div class="w-full flex justify-between *:w-full">
                <SudokuTimer class="text-left" ref="timerRef" :savedSeconds="seconds" @seconds="(sec) => handleSenconds(sec)" />
                <SudokuScore ref="scoreRef" :seconds />
                <p class="text-right">Mistakes: {{ mistakes }}/3</p>
            </div>
            <div class="w-full max-w-xl aspect-square">
                <table class="w-full h-full table-fixed text-2xl">
                    <tr class="sudoku-number" v-for="(line, lineIndex) in sudoku">
                        <td 
                            v-for="(numberPosition, colIndex) in line"
                            @click="(ev) => selectCell(ev,numberPosition)" 
                            class="max-w-[50px] max-h-[50px] relative" 
                            :class="[
                                numberPosition.color,
                                selectedRow === lineIndex ? 'bg-primary-light' : '',
                                selectedCol === colIndex ? 'bg-primary-light' : '',
                                selectedPosition === numberPosition ? 'bg-primary-dark!' : '',
                                selectedSquare && 
                                Math.floor(lineIndex / 3) === selectedSquare.row &&
                                Math.floor(colIndex / 3) === selectedSquare.col 
                                    ? 'bg-primary-light' 
                                    : ''
                                
                            ]"
                        >
                            <div
                                v-show="numberPosition.number === null"
                                class="note-number absolute h-full w-full top-0 left-0 grid grid-cols-3 grid-rows-3 text-sm"
                            >
                                <div v-for="n in 9" :key="n">{{ numberPosition.noteNumber.find(num=>num===n-1) !== undefined ? n : '' }}</div>
                                <!-- <div class="1"></div>
                                <div class="2"></div>
                                <div class="3"></div>
                                <div class="4"></div>
                                <div class="5"></div>
                                <div class="6"></div>
                                <div class="7"></div>
                                <div class="8"></div>
                                <div class="9"></div> -->
                            </div>
                            <div :class="[numberPosition.number === null ? 'opacity-0' : '']" >
                                {{ (numberPosition.number || 0) + 1 }}
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
                <button
                    v-for="value in [0,1,2,3,4,5,6,7,8]"
                    @click="addNumber(value)"
                    :class="[notesToggle ? 'bg-secondary-extra-light!  text-secondary!' : '']"
                >
                    {{ value + 1 }}
                </button>
                <button 
                    @click="notesToggle=!notesToggle"
                    :class="[notesToggle ? 'bg-primary-dark!' : '']"
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
    noteNumber: number[] // gray small numbers
}

const alertButtons = ['Try again'];
const isOpen = ref(false);
const sudokuGen = ref<Array<any>>(makepuzzle())
const sudokuSolved = ref<Array<any>>(solvepuzzle(sudokuGen))
const sudokuUserSolved = ref<Array<number|null>>([])
const answers = ref(0)
const mistakes = ref(0)

const timerRef = ref<InstanceType<typeof SudokuTimer> | null>(null)
const scoreRef = ref<InstanceType<typeof SudokuScore> | null>(null)
const seconds = ref(0)
const notesToggle = ref(false)

const sudoku = ref<Array<Array<NumberPosition>>>([])
const selectedElement = ref<HTMLElement | null>(null)
const selectedPosition = ref<NumberPosition | null>(null)

const answersColor = 'text-primary'
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

const cleanNoteNumber = (correctNumber: number) => {    
    sudoku.value.forEach((line,rowIndex) => {
        line.forEach((number,colIndex) => {
            if (rowIndex === selectedRow.value) {
                number.noteNumber = number.noteNumber.filter(noteNumber => noteNumber != correctNumber)
            }

            if (colIndex === selectedCol.value) {
                number.noteNumber = number.noteNumber.filter(noteNumber => noteNumber != correctNumber)
            }

            if (!selectedSquare.value) return

            const squareRowIndex = Math.floor(rowIndex / 3)
            const squareColIndex = Math.floor(colIndex / 3)
            if(
                squareRowIndex === selectedSquare.value.row &&
                squareColIndex === selectedSquare.value.col
            ) {
                number.noteNumber = number.noteNumber.filter(noteNumber => noteNumber != correctNumber)
            }
        })
    })
}

const startGame = () => {
    loadSudoku()
    if (!timerRef.value) return
    const sudokuLS = localStorage.getItem('sudoku')
    const sudokuUserSolvedLS = localStorage.getItem('sudokuUserSolved')    

    let sudoku = makepuzzle()

    if (sudokuLS) sudoku = sudokuStringtoArray(sudokuLS) 

    // default values or load last game
    mistakes.value = Number(localStorage.getItem('mistakes')) | 0
    answers.value = Number(localStorage.getItem('answers')) | 0
    seconds.value = Number(localStorage.getItem('seconds')) | 0
    sudokuUserSolved.value = sudokuUserSolvedLS ? sudokuStringtoArray(sudokuUserSolvedLS) : []    
    timerRef.value.start(seconds.value)
    

    generateSudoku(sudoku, sudokuUserSolved.value)
}

onMounted(()=> {
    startGame()    
})

const sudokuStringtoArray = (sudoku:string): Array<number|null> => {
    const sudokuArray = sudoku.split(',')        
    const sudokuNum = sudokuArray.map((str)=> str === '' ? null : Number(str))
    return sudokuNum
}

const handleSenconds = (secondsTimer:number) => {
    localStorage.setItem('seconds', seconds.value+'')
    seconds.value = secondsTimer
}

const generateSudoku = (sudokuG: Array<number | null>, sudokuUserSoluved?: Array<number | null>) => {
    sudokuGen.value = sudokuG
    localStorage.setItem('sudoku',sudokuGen.value.toString())    
    sudokuSolved.value = solvepuzzle(sudokuGen.value)
    
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
            let newNumber: NumberPosition = {number:number, index: indexSudoku, color:'', noteNumber:[]}

            if (notesNumber.value && notesNumber.value[indexSudoku]) {
                const notesNumbersArray = notesNumber.value[indexSudoku].notes
                newNumber.noteNumber = notesNumbersArray
            }
            if (sudokuUserSoluved && Number.isInteger(sudokuUserSoluved[indexSudoku])) {
                const userSolvedNum = sudokuUserSoluved[indexSudoku]
                newNumber.number = userSolvedNum
                newNumber.color = answersColor
            }



            newLine.push(newNumber)
            indexSudoku++ 
        }
        sudoku.value.push(newLine)
    }
}

const closeAlert = () => {
    isOpen.value = false

    // clean data
    localStorage.removeItem('score')
    localStorage.removeItem('mistakes')
    localStorage.removeItem('sudoku')
    localStorage.removeItem('seconds')
    localStorage.removeItem('sudokuUserSolved')
    localStorage.removeItem('sudokuState')

    startGame()
};

const selectCell = (ev:PointerEvent, numberPosition: NumberPosition) => {
    const td = ev.currentTarget as HTMLElement
    selectedElement.value = td
    selectedPosition.value = numberPosition
}

const addNumber = (number: number) => {
    if (!selectedPosition.value) return
    if (!selectedElement.value) return
    if (!scoreRef.value) return
    
    const correctNumber = sudokuSolved.value[selectedPosition.value.index]
    
    if (selectedPosition.value.number === correctNumber) return

    if (selectedPosition.value.number !== 0) selectedPosition.value.number = null

    if (notesToggle.value) {
        if (selectedPosition.value.noteNumber.find(num => num === number) !== undefined) {
            // remove number
            selectedPosition.value.noteNumber = selectedPosition.value.noteNumber.filter(num => num !== number);
        } else {
            // add number
            selectedPosition.value.noteNumber.push(number)
        }
        
        saveSudoku()

        return
    }

    if (number === correctNumber) {
        // correct
        selectedPosition.value.color = answersColor
        scoreRef.value.addScore()
        sudokuUserSolved.value[selectedPosition.value.index] = number
        localStorage.setItem('sudokuUserSolved',sudokuUserSolved.value.toString())
        cleanNoteNumber(number)
    } else {
        // mistake
        mistakes.value++
        if (mistakes.value>=3) {
            isOpen.value = true
            if (timerRef.value) timerRef.value.stop()
            return
        }
        scoreRef.value.addMistake()
        selectedPosition.value.color = mistakesColor
    }

    
    selectedPosition.value.number = number    
    
    saveSudoku()
}

const saveSudoku = () => {
    const toSave = sudoku.value.flat().map(cell => ({
      number: cell.number,
      notes: cell.noteNumber,
    }))
    localStorage.setItem('sudokuState', JSON.stringify(toSave))
}

const notesNumber = ref<null|{number:number,notes:number[]}[]>(null)

const loadSudoku = () => {
    const saved = localStorage.getItem('sudokuState')
    if (saved) {
        const parsed = JSON.parse(saved)
        notesNumber.value = parsed
    } else {
        notesNumber.value = null
    }
    
}


</script>

<style scoped>
    td {
        border:1px solid var(--secondary-light-color);
    }
    table {
        border:2px solid  var(--color-secondary);
        /* color: var(--terciary-dark-color); */
    }
    tr:nth-child(3n){
        border-bottom:2px solid var(--color-secondary);
    }

    td:nth-child(3n) {
        border-right:2px solid var(--color-secondary);
    }

    .note-number {
        color: var(--secondary-dark-color);
    }

    @media (min-width: 500px) {
        .sudoku-number{
            font-size: 30px;
        }
        .note-number {
            font-size: 16px;
        }
    }

    @media (max-width: 500px) {
        .sudoku-number{
            font-size: 22px;
        }
        .note-number {
            font-size: 12px;
        }
    }

    @media (max-width: 300px) {
        .sudoku-number{
            font-size: 16px;
        }
        .note-number {
            font-size: 8px;
        }
    }

    @media (max-width: 220px) {
        .sudoku-number{
            font-size: 14px;
        }
        .note-number {
            font-size: 7px;
        }
    }

    .buttons > button {
        /* border: solid 1px var(--color-primary); */
        border-radius: 25%;
        max-width: 2.5em;
        max-height: 2.5em;
        min-width: 2em;
        min-height: 2em;
        width: 2.5em;
        height: 2.5em;
        background-color: var(--primary-light-color);
        font-size: larger;
        color: var(--primary-color);
    }

    @media(hover: hover) {
        .buttons > button:hover{
            background-color: var(--primary-dark-color);
        }
    }

    .buttons > button:active{
        background-color: var(--primary-dark-color);
    }
</style>