<template>
  <div 
    class="max-w-2xl mx-auto p-5 text-center"
    tabindex="0"
    @keydown="handleKeydown"
  >
    <h1 class="text-3xl font-bold text-gray-800 mb-8">Sudoku Noter</h1>
    
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
        :disabled="!selectedCell"
      >
        Erase
      </button>
    </div>
    
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
            'border-b-4 border-b-gray-800': rowIndex === 2 || rowIndex === 5
          }"
          @click="selectCell(rowIndex, colIndex)"
        >
          <!-- Main number -->
          <span v-if="cell.value" class="text-xl font-bold">{{ cell.value }}</span>
          
          <!-- Center note -->
          <div v-if="cell.centerNote && !cell.value" class="text-xs text-gray-600 absolute inset-0 flex items-center justify-center">
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
        :disabled="!selectedCell"
      >
        {{ num }}
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

const sudokuGrid = ref(Array(9).fill().map(() => Array(9).fill().map(() => ({
  value: null,
  centerNote: [],
  cornerNotes: []
}))))
const selectedCell = ref(null)
const noteMode = ref(null) // null, 'center', or 'corners'
const history = ref([])

const selectCell = (row, col) => {
  selectedCell.value = { row, col }
}

const saveState = () => {
  const state = JSON.parse(JSON.stringify(sudokuGrid.value))
  history.value.push(state)
  if (history.value.length > 50) {
    history.value.shift()
  }
}

const undo = () => {
  if (history.value.length > 0) {
    sudokuGrid.value = history.value.pop()
  }
}

const toggleNoteMode = (mode) => {
  noteMode.value = noteMode.value === mode ? null : mode
}

const inputNumber = (number) => {
  if (!selectedCell.value) return
  
  const { row, col } = selectedCell.value
  const cell = sudokuGrid.value[row][col]
  
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
  if (!selectedCell.value) return
  
  const { row, col } = selectedCell.value
  const cell = sudokuGrid.value[row][col]
  
  saveState()
  
  cell.value = null
  cell.centerNote = []
  cell.cornerNotes = []
}

const submitPuzzle = () => {
  const puzzleData = sudokuGrid.value.map(row => row.map(cell => cell.value))
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
    eraseCell()
  }
}
</script>

