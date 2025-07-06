<template>
  <div class="container">
    <h1>Sudoku Noter</h1>
    <div class="sudoku-grid">
      <div
        v-for="(row, rowIndex) in sudokuGrid"
        :key="rowIndex"
        class="sudoku-row"
      >
        <div
          v-for="(cell, colIndex) in row"
          :key="colIndex"
          class="sudoku-cell"
          :class="{ 
            'selected': selectedCell?.row === rowIndex && selectedCell?.col === colIndex,
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
        :disabled="!selectedCell"
      >
        {{ num }}
      </button>
      <button
        class="clear-btn"
        @click="clearCell"
        :disabled="!selectedCell"
      >
        Clear
      </button>
    </div>
    
    <button class="submit-btn" @click="submitPuzzle">
      Start Playing
    </button>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const sudokuGrid = ref(Array(9).fill().map(() => Array(9).fill(null)))
const selectedCell = ref(null)

const selectCell = (row, col) => {
  selectedCell.value = { row, col }
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

onMounted(() => {
  document.addEventListener('keydown', (e) => {
    if (selectedCell.value && e.key >= '1' && e.key <= '9') {
      inputNumber(parseInt(e.key))
    } else if (selectedCell.value && (e.key === 'Backspace' || e.key === 'Delete')) {
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

.submit-btn {
  background-color: #4caf50;
  color: white;
  border: none;
  padding: 12px 30px;
  font-size: 16px;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.2s;
}

.submit-btn:hover {
  background-color: #45a049;
}
</style>