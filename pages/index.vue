<template>
  <div
    class="max-w-2xl mx-auto p-3 sm:p-5 text-center min-h-screen flex flex-col justify-center"
    tabindex="0"
    @keydown="handleKeydown"
  >
    <h1 class="text-2xl sm:text-3xl font-bold text-gray-800 mb-4 sm:mb-8">
      Sudoku Noter
    </h1>

    <p class="text-gray-600 mb-4 sm:mb-8 text-sm sm:text-base">
      Create your Sudoku puzzle and start solving it.
    </p>

    <div class="w-full max-w-sm sm:max-w-md mx-auto mb-4 sm:mb-8">
      <div
        class="grid grid-cols-9 border-2 sm:border-4 border-gray-800 bg-white aspect-square"
      >
        <div
          v-for="(cell, index) in sudokuGrid.flat()"
          :key="index"
          class="border border-gray-300 flex items-center justify-center text-sm sm:text-lg font-bold cursor-pointer bg-white transition-colors duration-200 hover:bg-gray-100 relative aspect-square"
          :class="{
            'bg-blue-100 border-2 border-blue-500':
              selectedCell?.row === Math.floor(index / 9) && selectedCell?.col === index % 9,
            'border-r-4 border-r-gray-800': (index % 9) === 2 || (index % 9) === 5,
            'border-b-4 border-b-gray-800': Math.floor(index / 9) === 2 || Math.floor(index / 9) === 5,
            'text-red-600 bg-red-50': cell && !isValidCell(Math.floor(index / 9), index % 9),
          }"
          @click="selectCell(Math.floor(index / 9), index % 9)"
        >
          {{ cell || "" }}
        </div>
      </div>
    </div>

    <div
      class="flex gap-2 sm:gap-3 justify-center mb-4 sm:mb-8 flex-wrap max-w-xs sm:max-w-none mx-auto"
    >
      <button
        v-for="num in 9"
        :key="num"
        class="w-8 h-8 sm:w-12 sm:h-12 border-2 border-gray-800 bg-white text-sm sm:text-lg font-bold cursor-pointer rounded transition-all duration-200 hover:bg-gray-100 disabled:opacity-50 disabled:cursor-not-allowed"
        @click="inputNumber(num)"
        :disabled="!selectedCell"
      >
        {{ num }}
      </button>
      <button
        class="w-8 h-8 sm:w-12 sm:h-12 border-2 border-gray-800 bg-red-50 text-red-700 text-sm sm:text-lg font-bold cursor-pointer rounded transition-all duration-200 hover:bg-red-100 disabled:opacity-50 disabled:cursor-not-allowed"
        @click="clearCell"
        :disabled="!selectedCell"
      >
        X
      </button>
    </div>

    <button
      class="bg-green-500 text-white px-6 py-2 sm:px-8 sm:py-3 text-base sm:text-lg rounded cursor-pointer transition-colors duration-200 hover:bg-green-600 border-none"
      @click="submitPuzzle"
    >
      Start Playing
    </button>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";

const sudokuGrid = ref(
  Array(9)
    .fill()
    .map(() => Array(9).fill(null))
);
const selectedCell = ref(null);

const selectCell = (row, col) => {
  selectedCell.value = { row, col };
};

const isValidCell = (row, col) => {
  const value = sudokuGrid.value[row][col];
  if (!value) return true;

  // Check row
  for (let c = 0; c < 9; c++) {
    if (c !== col && sudokuGrid.value[row][c] === value) {
      return false;
    }
  }

  // Check column
  for (let r = 0; r < 9; r++) {
    if (r !== row && sudokuGrid.value[r][col] === value) {
      return false;
    }
  }

  // Check 3x3 block
  const blockRow = Math.floor(row / 3) * 3;
  const blockCol = Math.floor(col / 3) * 3;

  for (let r = blockRow; r < blockRow + 3; r++) {
    for (let c = blockCol; c < blockCol + 3; c++) {
      if ((r !== row || c !== col) && sudokuGrid.value[r][c] === value) {
        return false;
      }
    }
  }

  return true;
};

const inputNumber = number => {
  if (selectedCell.value) {
    const { row, col } = selectedCell.value;
    sudokuGrid.value[row][col] = number;
    saveToLocalStorage();
  }
};

const clearCell = () => {
  if (selectedCell.value) {
    const { row, col } = selectedCell.value;
    sudokuGrid.value[row][col] = null;
    saveToLocalStorage();
  }
};

const saveToLocalStorage = () => {
  if (typeof window !== "undefined") {
    try {
      localStorage.setItem("sudokuInputGrid", JSON.stringify(sudokuGrid.value));
    } catch (e) {
      console.error("Failed to save to localStorage:", e);
    }
  }
};

const loadFromLocalStorage = () => {
  if (typeof window !== "undefined") {
    try {
      const saved = localStorage.getItem("sudokuInputGrid");
      if (saved) {
        const parsedGrid = JSON.parse(saved);
        sudokuGrid.value = parsedGrid;
      }
    } catch (e) {
      console.error("Failed to load from localStorage:", e);
    }
  }
};

const submitPuzzle = () => {
  const puzzleData = sudokuGrid.value.map(row => [...row]);
  const puzzleState = useState("sudokuPuzzle");
  puzzleState.value = puzzleData;

  // Save initial puzzle to localStorage
  if (typeof window !== "undefined") {
    try {
      localStorage.setItem("sudokuInitialPuzzle", JSON.stringify(puzzleData));
      localStorage.removeItem("sudokuGameState"); // Clear any previous game state
    } catch (e) {
      console.error("Failed to save puzzle to localStorage:", e);
    }
  }

  navigateTo("/play");
};

const handleKeydown = e => {
  if (selectedCell.value && e.key >= "1" && e.key <= "9") {
    inputNumber(parseInt(e.key));
  } else if (
    selectedCell.value &&
    (e.key === "Backspace" || e.key === "Delete")
  ) {
    clearCell();
  }
};

onMounted(() => {
  loadFromLocalStorage();
});
</script>
