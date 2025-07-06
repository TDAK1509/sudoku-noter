<template>
  <div 
    class="max-w-2xl mx-auto p-5 text-center"
    tabindex="0"
    @keydown="handleKeydown"
  >
    <h1 class="text-3xl font-bold text-gray-800 mb-4">Sudoku Game</h1>
    
    <!-- Timer -->
    <div class="text-2xl font-bold text-blue-600 mb-6">
      {{ formatTime(elapsedTime) }}
    </div>
    
    <!-- Control buttons -->
    <div class="flex gap-3 justify-center mb-6 flex-wrap">
      <button
        class="px-4 py-2 border-2 border-gray-800 bg-white text-sm font-bold cursor-pointer rounded transition-all duration-200 hover:bg-gray-100 disabled:opacity-50 disabled:cursor-not-allowed"
        @click="undo"
        :disabled="history.length === 0"
      >
        Undo
      </button>
      <button
        class="px-4 py-2 border-2 border-gray-800 text-sm font-bold cursor-pointer rounded transition-all duration-200"
        :class="noteMode === 'center' ? 'bg-blue-500 text-white' : 'bg-white hover:bg-gray-100'"
        @click="toggleNoteMode('center')"
      >
        Center Note
      </button>
      <button
        class="px-4 py-2 border-2 border-gray-800 text-sm font-bold cursor-pointer rounded transition-all duration-200"
        :class="noteMode === 'corners' ? 'bg-blue-500 text-white' : 'bg-white hover:bg-gray-100'"
        @click="toggleNoteMode('corners')"
      >
        Corners Note
      </button>
      <button
        class="px-4 py-2 border-2 border-gray-800 bg-red-50 text-red-700 text-sm font-bold cursor-pointer rounded transition-all duration-200 hover:bg-red-100 disabled:opacity-50 disabled:cursor-not-allowed"
        @click="eraseCell"
        :disabled="!selectedCell || isSelectedCellFixed()"
      >
        Erase
      </button>
    </div>
    
    <div class="inline-block border-4 border-gray-800 mb-8">
      <div
        v-for="(row, rowIndex) in gameGrid"
        :key="rowIndex"
        class="flex"
      >
        <div
          v-for="(cell, colIndex) in row"
          :key="colIndex"
          class="w-12 h-12 border border-gray-300 flex items-center justify-center text-xl font-bold cursor-pointer bg-white transition-colors duration-200 hover:bg-gray-100 relative"
          :class="{ 
            'bg-blue-100 border-2 border-blue-500': selectedCell?.row === rowIndex && selectedCell?.col === colIndex,
            'bg-gray-100 text-gray-800 font-black hover:bg-gray-100': isFixedCell(rowIndex, colIndex),
            'border-r-4 border-r-gray-800': colIndex === 2 || colIndex === 5,
            'border-b-4 border-b-gray-800': rowIndex === 2 || rowIndex === 5,
            'text-red-600 bg-red-50': cell.value && !isValidCell(rowIndex, colIndex)
          }"
          @click="selectCell(rowIndex, colIndex)"
        >
          <!-- Main number -->
          <span v-if="cell.value" class="text-xl font-bold" :class="{
            'text-red-600': cell.value && !isValidCell(rowIndex, colIndex),
            'text-blue-600': cell.value && isValidCell(rowIndex, colIndex) && !isFixedCell(rowIndex, colIndex),
            'text-gray-800': isFixedCell(rowIndex, colIndex)
          }">{{ cell.value }}</span>
          
          <!-- Center note -->
          <div v-if="cell.centerNote && cell.centerNote.length > 0 && !cell.value" class="text-xs text-gray-600 absolute inset-0 flex items-center justify-center">
            {{ cell.centerNote.join('') }}
          </div>
          
          <!-- Corner notes -->
          <div v-if="cell.cornerNotes && cell.cornerNotes.length > 0 && !cell.value" class="absolute inset-0 text-xs text-gray-600">
            <div class="absolute top-0 left-0 w-full h-1/2 flex flex-wrap justify-start items-start pl-0.5 pt-0.5">
              <span v-for="(note, index) in cell.cornerNotes.slice(0, 4)" :key="index" class="text-xs leading-none mr-0.5">{{ note }}</span>
            </div>
            <div v-if="cell.cornerNotes.length > 4" class="absolute bottom-0 left-0 w-full h-1/2 flex flex-wrap justify-start items-end pl-0.5 pb-0.5">
              <span v-for="(note, index) in cell.cornerNotes.slice(4)" :key="index" class="text-xs leading-none mr-0.5">{{ note }}</span>
            </div>
          </div>
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
import { ref, onMounted, onUnmounted } from 'vue'

const gameGrid = ref(Array(9).fill().map(() => Array(9).fill().map(() => ({
  value: null,
  centerNote: [],
  cornerNotes: []
}))))
const fixedCells = ref(Array(9).fill().map(() => Array(9).fill(false)))
const selectedCell = ref(null)
const noteMode = ref(null) // null, 'center', or 'corners'
const history = ref([])
const startTime = ref(Date.now())
const elapsedTime = ref(0)

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

const isValidCell = (row, col) => {
  const value = gameGrid.value[row][col].value
  if (!value) return true
  
  // Check row
  for (let c = 0; c < 9; c++) {
    if (c !== col && gameGrid.value[row][c].value === value) {
      return false
    }
  }
  
  // Check column
  for (let r = 0; r < 9; r++) {
    if (r !== row && gameGrid.value[r][col].value === value) {
      return false
    }
  }
  
  // Check 3x3 block
  const blockRow = Math.floor(row / 3) * 3
  const blockCol = Math.floor(col / 3) * 3
  
  for (let r = blockRow; r < blockRow + 3; r++) {
    for (let c = blockCol; c < blockCol + 3; c++) {
      if ((r !== row || c !== col) && gameGrid.value[r][c].value === value) {
        return false
      }
    }
  }
  
  return true
}

const saveState = () => {
  const state = JSON.parse(JSON.stringify(gameGrid.value))
  history.value.push(state)
  if (history.value.length > 50) {
    history.value.shift()
  }
}

const undo = () => {
  if (history.value.length > 0) {
    gameGrid.value = history.value.pop()
  }
}

const toggleNoteMode = (mode) => {
  noteMode.value = noteMode.value === mode ? null : mode
}

const inputNumber = (number) => {
  if (!selectedCell.value || isSelectedCellFixed()) return
  
  const { row, col } = selectedCell.value
  const cell = gameGrid.value[row][col]
  
  saveState()
  
  if (noteMode.value === 'center') {
    if (!cell.centerNote.includes(number)) {
      cell.centerNote.push(number)
      cell.centerNote.sort((a, b) => a - b)
    }
    cell.value = null
  } else if (noteMode.value === 'corners') {
    if (!cell.cornerNotes.includes(number)) {
      cell.cornerNotes.push(number)
      cell.cornerNotes.sort((a, b) => a - b)
    }
    cell.value = null
  } else {
    cell.value = number
    cell.centerNote = []
    cell.cornerNotes = []
  }
}

const eraseCell = () => {
  if (!selectedCell.value || isSelectedCellFixed()) return
  
  const { row, col } = selectedCell.value
  const cell = gameGrid.value[row][col]
  
  saveState()
  
  cell.value = null
  cell.centerNote = []
  cell.cornerNotes = []
}

const goBack = () => {
  navigateTo('/')
}

const resetGame = () => {
  saveState()
  
  for (let row = 0; row < 9; row++) {
    for (let col = 0; col < 9; col++) {
      if (!fixedCells.value[row][col]) {
        gameGrid.value[row][col] = {
          value: null,
          centerNote: [],
          cornerNotes: []
        }
      }
    }
  }
  selectedCell.value = null
}

const handleKeydown = (e) => {
  if (selectedCell.value && !isSelectedCellFixed() && e.key >= '1' && e.key <= '9') {
    inputNumber(parseInt(e.key))
  } else if (selectedCell.value && !isSelectedCellFixed() && (e.key === 'Backspace' || e.key === 'Delete')) {
    eraseCell()
  }
}

const formatTime = (seconds) => {
  const minutes = Math.floor(seconds / 60)
  const remainingSeconds = seconds % 60
  return `${minutes.toString().padStart(2, '0')}:${remainingSeconds.toString().padStart(2, '0')}`
}

let timerInterval = null

onMounted(() => {
  const route = useRoute()
  if (route.query.puzzle) {
    try {
      const puzzle = JSON.parse(route.query.puzzle)
      for (let row = 0; row < 9; row++) {
        for (let col = 0; col < 9; col++) {
          gameGrid.value[row][col] = {
            value: puzzle[row][col],
            centerNote: [],
            cornerNotes: []
          }
          fixedCells.value[row][col] = puzzle[row][col] !== null
        }
      }
    } catch (e) {
      console.error('Error parsing puzzle data:', e)
    }
  }
  
  // Start timer
  startTime.value = Date.now()
  timerInterval = setInterval(() => {
    elapsedTime.value = Math.floor((Date.now() - startTime.value) / 1000)
  }, 1000)
})

onUnmounted(() => {
  if (timerInterval) {
    clearInterval(timerInterval)
  }
})
</script>

