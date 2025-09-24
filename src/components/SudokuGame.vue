<template>
  <div id="container" class="flex flex-col gap-12 justify-center text-center p-10 content-center h-full">
    <!-- <p>puzzle = {{ sudoku }}</p>
    <div class="flex">
        <div class="sudoku-number" v-for="(line) in sudokuLines">
            <div class="p-2" v-for="number in line">{{ number }}</div>
        </div>

    </div> -->

    <table class="bg-gray-900">
        <tr class="sudoku-number" v-for="(line) in sudokuLines">
            <td @click="(ev) => selectCell(ev,numberPosition)" class="w-4! h-10!" :class="numberPosition.number === null ? 'bg-gray-800' : ''" v-for="numberPosition in line">{{ numberPosition.number === null ? ' ' : numberPosition.number + 1 }}</td>
        </tr>

    </table>

    <!-- <br v-if="index%9===0"> -->
    <!-- <p>puzzleSolve = {{ puzzleSolve }}</p>
    <p>puzzleRate = {{ puzzleRate }}</p> -->
    
      <div class="flex justify-between w-full">
        <button @click="addNumber(value)" class="border! p-2! rounded! hover:bg-gray-700! active:bg-gray-700!" v-for="value in [1,2,3,4,5,6,7,8,9]">{{ value }}</button>
      </div>
  </div>
</template>

<script setup lang="ts">
import { makepuzzle, solvepuzzle, ratepuzzle } from "sudoku";
const sudoku: Array<any> = makepuzzle()
const sudokuSolved: Array<any> = solvepuzzle(sudoku)
interface NumberPosition {
    number: number | null,
    index: number
}

const sudokuLines:Array<Array<NumberPosition>> = []
let indexSudoku = 0
// 9 lines
for (let line = 0; line < 9; line++) {
    const newLine = []
    // 9 numbers
    for (let index = 0; index < 9; index++) {
        const number = sudoku[indexSudoku]
        newLine.push({number:number, index: indexSudoku})
        indexSudoku++ 
    }
    sudokuLines.push(newLine)
}


let selectedElement: HTMLElement | null = null
let selectedPosition: NumberPosition | null = null

const selectCell = (ev:PointerEvent, numberPosition: NumberPosition) => {
    const td = ev.currentTarget as HTMLElement
    td.classList.add('bg-gray-700!')
    selectedElement?.classList.remove('bg-gray-700!')
    selectedElement = td
    selectedPosition = numberPosition
}

const addNumber = (number: number) => {
    if (!selectedPosition) return
    if (!selectedElement) return

    selectedElement.textContent = number + ''

    if (number === sudokuSolved[selectedPosition.index]+1) {
        selectedElement.classList.add('text-gray-400!')
    } else {
        selectedElement.classList.add('text-red-400')
    }
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
