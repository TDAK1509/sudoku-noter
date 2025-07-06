<template>
  <div 
    class="max-w-2xl mx-auto p-5 text-center"
    tabindex="0"
    @keydown="handleKeydown"
  >
    <h1 class="text-3xl font-bold text-gray-800 mb-8">Sudoku Noter</h1>
    
    <div class="inline-block border-4 border-gray-800 mb-8">
      <div
        v-for="(row, rowIndex) in sudokuGrid"
        :key="rowIndex"
        class="flex"
      >
        <div
          v-for="(cell, colIndex) in row"
          :key="colIndex"
          class="w-12 h-12 border border-gray-300 flex items-center justify-center text-xl font-bold cursor-pointer bg-white transition-colors duration-200 hover:bg-gray-100 relative"
          :class="{ 
            'bg-blue-100 border-2 border-blue-500': selectedCell?.row === rowIndex && selectedCell?.col === colIndex,
            'border-r-4 border-r-gray-800': colIndex === 2 || colIndex === 5,
            'border-b-4 border-b-gray-800': rowIndex === 2 || rowIndex === 5,
            'text-red-600 bg-red-50': cell && !isValidCell(rowIndex, colIndex)
          }"
          @click="selectCell(rowIndex, colIndex)"
        >
          {{ cell || '' }}
        </div>
      </div>
    </div>
    
    <div class="flex gap-3 justify-center mb-8 flex-wrap">
      <button
        v-for="num in 9"
        :key="num"
        class="w-12 h-12 border-2 border-gray-800 bg-white text-lg font-bold cursor-pointer rounded transition-all duration-200 hover:bg-gray-100 disabled:opacity-50 disabled:cursor-not-allowed"
        @click="inputNumber(num)"
        :disabled="!selectedCell"
      >
        {{ num }}
      </button>
      <button
        class="w-12 h-12 border-2 border-gray-800 bg-red-50 text-red-700 font-bold cursor-pointer rounded transition-all duration-200 hover:bg-red-100 disabled:opacity-50 disabled:cursor-not-allowed"
        @click="clearCell"
        :disabled="!selectedCell"
      >
        X
      </button>
    </div>
    
    <button 
      class="bg-green-500 text-white px-8 py-3 text-lg rounded cursor-pointer transition-colors duration-200 hover:bg-green-600 border-none"
      @click="submitPuzzle"
    >
      Start Playing
    </button>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const sudokuGrid = ref(Array(9).fill().map(() => Array(9).fill(null)))
const selectedCell = ref(null)

const selectCell = (row, col) => {
  selectedCell.value = { row, col }
}

const isValidCell = (row, col) => {
  const value = sudokuGrid.value[row][col]
  if (!value) return true
  
  // Check row
  for (let c = 0; c < 9; c++) {
    if (c !== col && sudokuGrid.value[row][c] === value) {
      return false
    }
  }
  
  // Check column
  for (let r = 0; r < 9; r++) {
    if (r !== row && sudokuGrid.value[r][col] === value) {
      return false
    }
  }
  
  // Check 3x3 block
  const blockRow = Math.floor(row / 3) * 3
  const blockCol = Math.floor(col / 3) * 3
  
  for (let r = blockRow; r < blockRow + 3; r++) {
    for (let c = blockCol; c < blockCol + 3; c++) {
      if ((r !== row || c !== col) && sudokuGrid.value[r][c] === value) {
        return false
      }
    }
  }
  
  return true
}

const inputNumber = (number) => {
  if (selectedCell.value) {
    const { row, col } = selectedCell.value
    sudokuGrid.value[row][col] = number
  }
}

const clearCell = () => {
  if (selectedCell.value) {
    const { row, col } = selectedCell.value
    sudokuGrid.value[row][col] = null
  }
}

const submitPuzzle = () => {
  const puzzleData = sudokuGrid.value.map(row => [...row])
  navigateTo({
    path: '/play',
    query: {
      puzzle: JSON.stringify(puzzleData)
    }
  })
}

const handleKeydown = (e) => {
  if (selectedCell.value && e.key >= '1' && e.key <= '9') {
    inputNumber(parseInt(e.key))
  } else if (selectedCell.value && (e.key === 'Backspace' || e.key === 'Delete')) {
    clearCell()
  }
}
</script>

