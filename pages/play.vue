<template>
  <div class="container">
    <h1>Sudoku Game</h1>
    <div class="sudoku-grid">
      <div
        v-for="(row, rowIndex) in gameGrid"
        :key="rowIndex"
        class="sudoku-row"
      >
        <div
          v-for="(cell, colIndex) in row"
          :key="colIndex"
          class="sudoku-cell"
          :class="{ 
            'selected': selectedCell?.row === rowIndex && selectedCell?.col === colIndex,
            'fixed': isFixedCell(rowIndex, colIndex),
            'thick-border-right': colIndex === 2 || colIndex === 5,
            'thick-border-bottom': rowIndex === 2 || rowIndex === 5
          }"
          @click="selectCell(rowIndex, colIndex)"
        >
          {{ cell || '' }}
        </div>
      </div>
    </div>
    
    <div class="number-buttons">
      <button
        v-for="num in 9"
        :key="num"
        class="number-btn"
        @click="inputNumber(num)"
        :disabled="!selectedCell || isSelectedCellFixed()"
      >
        {{ num }}
      </button>
      <button
        class="clear-btn"
        @click="clearCell"
        :disabled="!selectedCell || isSelectedCellFixed()"
      >
        Clear
      </button>
    </div>
    
    <div class="game-actions">
      <button class="back-btn" @click="goBack">
        Back to Input
      </button>
      <button class="reset-btn" @click="resetGame">
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
  
  document.addEventListener('keydown', (e) => {
    if (selectedCell.value && !isSelectedCellFixed() && e.key >= '1' && e.key <= '9') {
      inputNumber(parseInt(e.key))
    } else if (selectedCell.value && !isSelectedCellFixed() && (e.key === 'Backspace' || e.key === 'Delete')) {
      clearCell()
    }
  })
})
</script>

<style scoped>
.container {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
  text-align: center;
}

h1 {
  color: #333;
  margin-bottom: 30px;
}

.sudoku-grid {
  display: inline-block;
  border: 3px solid #333;
  margin-bottom: 30px;
}

.sudoku-row {
  display: flex;
}

.sudoku-cell {
  width: 50px;
  height: 50px;
  border: 1px solid #ccc;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 20px;
  font-weight: bold;
  cursor: pointer;
  background: white;
  transition: background-color 0.2s;
}

.sudoku-cell:hover {
  background-color: #f0f0f0;
}

.sudoku-cell.selected {
  background-color: #e3f2fd;
  border: 2px solid #2196f3;
}

.sudoku-cell.fixed {
  background-color: #f5f5f5;
  color: #333;
  font-weight: 900;
}

.sudoku-cell.fixed:hover {
  background-color: #f5f5f5;
}

.sudoku-cell.thick-border-right {
  border-right: 3px solid #333;
}

.sudoku-cell.thick-border-bottom {
  border-bottom: 3px solid #333;
}

.number-buttons {
  display: flex;
  gap: 10px;
  justify-content: center;
  margin-bottom: 30px;
  flex-wrap: wrap;
}

.number-btn, .clear-btn {
  width: 50px;
  height: 50px;
  border: 2px solid #333;
  background: white;
  font-size: 18px;
  font-weight: bold;
  cursor: pointer;
  border-radius: 4px;
  transition: all 0.2s;
}

.number-btn:hover, .clear-btn:hover {
  background-color: #f0f0f0;
}

.number-btn:disabled, .clear-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.clear-btn {
  background-color: #ffebee;
  color: #d32f2f;
}

.game-actions {
  display: flex;
  gap: 20px;
  justify-content: center;
}

.back-btn, .reset-btn {
  padding: 12px 30px;
  font-size: 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.2s;
}

.back-btn {
  background-color: #757575;
  color: white;
}

.back-btn:hover {
  background-color: #616161;
}

.reset-btn {
  background-color: #ff9800;
  color: white;
}

.reset-btn:hover {
  background-color: #f57c00;
}
</style>