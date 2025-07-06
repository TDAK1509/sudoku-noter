<template>
  <div 
    class="max-w-2xl mx-auto p-5 text-center"
    tabindex="0"
    @keydown="handleKeydown"
  >
    <h1 class="text-3xl font-bold text-gray-800 mb-8">Sudoku Game</h1>
    <div class="inline-block border-4 border-gray-800 mb-8">
      <div
        v-for="(row, rowIndex) in gameGrid"
        :key="rowIndex"
        class="flex"
      >
        <div
          v-for="(cell, colIndex) in row"
          :key="colIndex"
          class="w-12 h-12 border border-gray-300 flex items-center justify-center text-xl font-bold cursor-pointer bg-white transition-colors duration-200 hover:bg-gray-100"
          :class="{ 
            'bg-blue-100 border-2 border-blue-500': selectedCell?.row === rowIndex && selectedCell?.col === colIndex,
            'bg-gray-100 text-gray-800 font-black hover:bg-gray-100': isFixedCell(rowIndex, colIndex),
            'border-r-4 border-r-gray-800': colIndex === 2 || colIndex === 5,
            'border-b-4 border-b-gray-800': rowIndex === 2 || rowIndex === 5
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
        :disabled="!selectedCell || isSelectedCellFixed()"
      >
        {{ num }}
      </button>
      <button
        class="w-12 h-12 border-2 border-gray-800 bg-red-50 text-red-700 font-bold cursor-pointer rounded transition-all duration-200 hover:bg-red-100 disabled:opacity-50 disabled:cursor-not-allowed"
        @click="clearCell"
        :disabled="!selectedCell || isSelectedCellFixed()"
      >
        X
      </button>
    </div>
    
    <div class="flex gap-5 justify-center">
      <button 
        class="bg-gray-600 text-white px-8 py-3 text-lg rounded cursor-pointer transition-colors duration-200 hover:bg-gray-700 border-none"
        @click="goBack"
      >
        Back to Input
      </button>
      <button 
        class="bg-orange-500 text-white px-8 py-3 text-lg rounded cursor-pointer transition-colors duration-200 hover:bg-orange-600 border-none"
        @click="resetGame"
      >
        Reset Game
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const gameGrid = ref(Array(9).fill().map(() => Array(9).fill(null)))
const fixedCells = ref(Array(9).fill().map(() => Array(9).fill(false)))
const selectedCell = ref(null)

const selectCell = (row, col) => {
  selectedCell.value = { row, col }
}

const isFixedCell = (row, col) => {
  return fixedCells.value[row][col]
}

const isSelectedCellFixed = () => {
  if (!selectedCell.value) return false
  const { row, col } = selectedCell.value
  return fixedCells.value[row][col]
}

const inputNumber = (number) => {
  if (selectedCell.value && !isSelectedCellFixed()) {
    const { row, col } = selectedCell.value
    gameGrid.value[row][col] = number
  }
}

const clearCell = () => {
  if (selectedCell.value && !isSelectedCellFixed()) {
    const { row, col } = selectedCell.value
    gameGrid.value[row][col] = null
  }
}

const goBack = () => {
  navigateTo('/')
}

const resetGame = () => {
  for (let row = 0; row < 9; row++) {
    for (let col = 0; col < 9; col++) {
      if (!fixedCells.value[row][col]) {
        gameGrid.value[row][col] = null
      }
    }
  }
  selectedCell.value = null
}

const handleKeydown = (e) => {
  if (selectedCell.value && !isSelectedCellFixed() && e.key >= '1' && e.key <= '9') {
    inputNumber(parseInt(e.key))
  } else if (selectedCell.value && !isSelectedCellFixed() && (e.key === 'Backspace' || e.key === 'Delete')) {
    clearCell()
  }
}

onMounted(() => {
  const route = useRoute()
  if (route.query.puzzle) {
    try {
      const puzzle = JSON.parse(route.query.puzzle)
      for (let row = 0; row < 9; row++) {
        for (let col = 0; col < 9; col++) {
          gameGrid.value[row][col] = puzzle[row][col]
          fixedCells.value[row][col] = puzzle[row][col] !== null
        }
      }
    } catch (e) {
      console.error('Error parsing puzzle data:', e)
    }
  }
})
</script>

