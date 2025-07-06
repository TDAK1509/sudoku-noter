<template>
  <div
    class="max-w-2xl mx-auto p-3 sm:p-5 text-center min-h-screen flex flex-col justify-center"
    tabindex="0"
    @keydown="handleKeydown"
  >
    <h1 class="text-2xl sm:text-3xl font-bold text-gray-800 mb-2 sm:mb-4">
      Sudoku Game
    </h1>

    <!-- Timer -->
    <div class="text-xl sm:text-2xl font-bold text-blue-600 mb-4 sm:mb-6">
      {{ formatTime(elapsedTime) }}
    </div>

    <!-- Control buttons -->
    <div class="flex gap-2 sm:gap-3 justify-center mb-4 sm:mb-6 flex-wrap">
      <button
        class="px-2 py-1 sm:px-4 sm:py-2 border-2 border-gray-800 bg-white text-xs sm:text-sm font-bold cursor-pointer rounded transition-all duration-200 hover:bg-gray-100 disabled:opacity-50 disabled:cursor-not-allowed"
        @click="undo"
        :disabled="history.length === 0 || isGameComplete"
      >
        Undo
      </button>
      <button
        class="px-2 py-1 sm:px-4 sm:py-2 border-2 border-gray-800 text-xs sm:text-sm font-bold cursor-pointer rounded transition-all duration-200 disabled:opacity-50 disabled:cursor-not-allowed"
        :class="
          noteMode === 'center'
            ? 'bg-blue-500 text-white'
            : 'bg-white hover:bg-gray-100'
        "
        @click="toggleNoteMode('center')"
        :disabled="isGameComplete"
      >
        Center Note
      </button>
      <button
        class="px-2 py-1 sm:px-4 sm:py-2 border-2 border-gray-800 text-xs sm:text-sm font-bold cursor-pointer rounded transition-all duration-200 disabled:opacity-50 disabled:cursor-not-allowed"
        :class="
          noteMode === 'corners'
            ? 'bg-blue-500 text-white'
            : 'bg-white hover:bg-gray-100'
        "
        @click="toggleNoteMode('corners')"
        :disabled="isGameComplete"
      >
        Corners Note
      </button>
      <button
        class="px-2 py-1 sm:px-4 sm:py-2 border-2 border-gray-800 bg-red-50 text-red-700 text-xs sm:text-sm font-bold cursor-pointer rounded transition-all duration-200 hover:bg-red-100 disabled:opacity-50 disabled:cursor-not-allowed"
        @click="eraseCell"
        :disabled="!selectedCell || isSelectedCellFixed() || isGameComplete"
      >
        Erase
      </button>
    </div>

    <div class="w-full max-w-sm sm:max-w-md mx-auto mb-4 sm:mb-8">
      <div
        class="grid grid-cols-9 border-2 sm:border-4 border-gray-800 bg-white aspect-square"
      >
        <div
          v-for="(cell, index) in gameGrid.flat()"
          :key="index"
          class="border border-gray-300 flex items-center justify-center text-sm sm:text-lg font-bold transition-colors duration-200 relative aspect-square"
          :class="{
            'bg-blue-100 border-2 border-blue-500':
              selectedCell?.row === Math.floor(index / 9) &&
              selectedCell?.col === index % 9 &&
              !isGameComplete,
            'bg-gray-100 text-gray-800 font-black hover:bg-gray-100':
              isFixedCell(Math.floor(index / 9), index % 9),
            'border-r-4 border-r-gray-800': (index % 9) === 2 || (index % 9) === 5,
            'border-b-4 border-b-gray-800': Math.floor(index / 9) === 2 || Math.floor(index / 9) === 5,
            'text-red-600 bg-red-50':
              cell.value && !isValidCell(Math.floor(index / 9), index % 9),
            'cursor-pointer hover:bg-gray-100': !isGameComplete,
            'cursor-not-allowed': isGameComplete,
          }"
          @click="!isGameComplete && selectCell(Math.floor(index / 9), index % 9)"
        >
          <!-- Main number -->
          <span
            v-if="cell.value"
            class="text-sm sm:text-lg font-bold"
            :class="{
              'text-red-600': cell.value && !isValidCell(Math.floor(index / 9), index % 9),
              'text-blue-600':
                cell.value &&
                isValidCell(Math.floor(index / 9), index % 9) &&
                !isFixedCell(Math.floor(index / 9), index % 9),
              'text-gray-800': isFixedCell(Math.floor(index / 9), index % 9),
            }"
            >{{ cell.value }}</span
          >

          <!-- Center note -->
          <div
            v-if="cell.centerNote && cell.centerNote.length > 0 && !cell.value"
            class="text-[10px] sm:text-xs text-gray-600 absolute inset-0 flex items-center justify-center"
          >
            {{ cell.centerNote.join("") }}
          </div>

          <!-- Corner notes -->
          <div
            v-if="
              cell.cornerNotes && cell.cornerNotes.length > 0 && !cell.value
            "
            class="absolute inset-0 text-[8px] sm:text-[10px] text-gray-600"
          >
            <div
              class="absolute top-0 left-0 w-full h-1/2 flex flex-wrap justify-start items-start pl-0.5 pt-0.5"
            >
              <span
                v-for="(note, noteIndex) in cell.cornerNotes.slice(0, 4)"
                :key="noteIndex"
                class="text-[8px] sm:text-[10px] leading-none mr-0.5"
                >{{ note }}</span
              >
            </div>
            <div
              v-if="cell.cornerNotes.length > 4"
              class="absolute bottom-0 left-0 w-full h-1/2 flex flex-wrap justify-start items-end pl-0.5 pb-0.5"
            >
              <span
                v-for="(note, noteIndex) in cell.cornerNotes.slice(4)"
                :key="noteIndex"
                class="text-[8px] sm:text-[10px] leading-none mr-0.5"
                >{{ note }}</span
              >
            </div>
          </div>
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
        :disabled="!selectedCell || isSelectedCellFixed() || isGameComplete"
      >
        {{ num }}
      </button>
    </div>

    <div class="flex gap-3 sm:gap-5 justify-center flex-wrap">
      <button
        class="bg-gray-600 text-white px-4 py-2 sm:px-8 sm:py-3 text-sm sm:text-lg rounded cursor-pointer transition-colors duration-200 hover:bg-gray-700 border-none disabled:opacity-50 disabled:cursor-not-allowed"
        @click="goBack"
        :disabled="isGameComplete"
      >
        Back to Input
      </button>
      <button
        class="bg-orange-500 text-white px-4 py-2 sm:px-8 sm:py-3 text-sm sm:text-lg rounded cursor-pointer transition-colors duration-200 hover:bg-orange-600 border-none disabled:opacity-50 disabled:cursor-not-allowed"
        @click="resetGame"
        :disabled="isGameComplete"
      >
        Reset Game
      </button>
    </div>

    <!-- Congratulation Modal -->
    <div
      v-if="isGameComplete"
      class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50"
    >
      <div class="bg-white rounded-lg p-8 max-w-md mx-4 text-center">
        <div class="text-6xl mb-4">🎉</div>
        <h2 class="text-2xl font-bold text-gray-800 mb-4">Congratulations!</h2>
        <p class="text-gray-600 mb-4">
          You've successfully completed the Sudoku puzzle!
        </p>
        <p class="text-lg font-semibold text-blue-600 mb-6">
          Time: {{ formatTime(elapsedTime) }}
        </p>
        <button
          class="bg-green-500 text-white px-8 py-3 text-lg rounded cursor-pointer transition-colors duration-200 hover:bg-green-600 border-none"
          @click="startNewGame"
        >
          Start New Game
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from "vue";

const gameGrid = ref(
  Array(9)
    .fill()
    .map(() =>
      Array(9)
        .fill()
        .map(() => ({
          value: null,
          centerNote: [],
          cornerNotes: [],
        }))
    )
);
const fixedCells = ref(
  Array(9)
    .fill()
    .map(() => Array(9).fill(false))
);
const selectedCell = ref(null);
const noteMode = ref(null); // null, 'center', or 'corners'
const history = ref([]);
const startTime = ref(Date.now());
const elapsedTime = ref(0);
const isGameComplete = ref(false);

const selectCell = (row, col) => {
  selectedCell.value = { row, col };
};

const isFixedCell = (row, col) => {
  return fixedCells.value[row][col];
};

const isSelectedCellFixed = () => {
  if (!selectedCell.value) return false;
  const { row, col } = selectedCell.value;
  return fixedCells.value[row][col];
};

const isValidCell = (row, col) => {
  const value = gameGrid.value[row][col].value;
  if (!value) return true;

  // Check row
  for (let c = 0; c < 9; c++) {
    if (c !== col && gameGrid.value[row][c].value === value) {
      return false;
    }
  }

  // Check column
  for (let r = 0; r < 9; r++) {
    if (r !== row && gameGrid.value[r][col].value === value) {
      return false;
    }
  }

  // Check 3x3 block
  const blockRow = Math.floor(row / 3) * 3;
  const blockCol = Math.floor(col / 3) * 3;

  for (let r = blockRow; r < blockRow + 3; r++) {
    for (let c = blockCol; c < blockCol + 3; c++) {
      if ((r !== row || c !== col) && gameGrid.value[r][c].value === value) {
        return false;
      }
    }
  }

  return true;
};

const checkGameComplete = () => {
  // Check if all cells are filled
  for (let row = 0; row < 9; row++) {
    for (let col = 0; col < 9; col++) {
      if (!gameGrid.value[row][col].value) {
        return false;
      }
    }
  }

  // Check if all cells are valid
  for (let row = 0; row < 9; row++) {
    for (let col = 0; col < 9; col++) {
      if (!isValidCell(row, col)) {
        return false;
      }
    }
  }

  // Game is complete!
  isGameComplete.value = true;
  if (timerInterval) {
    clearInterval(timerInterval);
  }
  return true;
};

const saveToLocalStorage = () => {
  if (typeof window !== "undefined") {
    try {
      const gameState = {
        gameGrid: gameGrid.value,
        fixedCells: fixedCells.value,
        startTime: startTime.value,
        elapsedTime: elapsedTime.value,
        isGameComplete: isGameComplete.value,
        noteMode: noteMode.value,
      };
      localStorage.setItem("sudokuGameState", JSON.stringify(gameState));
    } catch (e) {
      console.error("Failed to save game state to localStorage:", e);
    }
  }
};

const loadFromLocalStorage = () => {
  if (typeof window !== "undefined") {
    try {
      const savedGameState = localStorage.getItem("sudokuGameState");
      if (savedGameState) {
        const gameState = JSON.parse(savedGameState);
        gameGrid.value = gameState.gameGrid;
        fixedCells.value = gameState.fixedCells;
        startTime.value = gameState.startTime;
        elapsedTime.value = gameState.elapsedTime;
        isGameComplete.value = gameState.isGameComplete;
        noteMode.value = gameState.noteMode || null;
        return true;
      }
    } catch (e) {
      console.error("Failed to load game state from localStorage:", e);
    }
  }
  return false;
};

const startNewGame = () => {
  // Clear localStorage
  if (typeof window !== "undefined") {
    try {
      localStorage.removeItem("sudokuGameState");
      localStorage.removeItem("sudokuInitialPuzzle");
      localStorage.removeItem("sudokuInputGrid");
    } catch (e) {
      console.error("Failed to clear localStorage:", e);
    }
  }
  navigateTo("/");
};

const saveState = () => {
  const state = JSON.parse(JSON.stringify(gameGrid.value));
  history.value.push(state);
  if (history.value.length > 50) {
    history.value.shift();
  }
};

const undo = () => {
  if (history.value.length > 0 && !isGameComplete.value) {
    gameGrid.value = history.value.pop();
    saveToLocalStorage();
  }
};

const toggleNoteMode = mode => {
  noteMode.value = noteMode.value === mode ? null : mode;
};

const inputNumber = number => {
  if (!selectedCell.value || isSelectedCellFixed() || isGameComplete.value)
    return;

  const { row, col } = selectedCell.value;
  const cell = gameGrid.value[row][col];

  saveState();

  if (noteMode.value === "center") {
    if (!cell.centerNote.includes(number)) {
      cell.centerNote.push(number);
      cell.centerNote.sort((a, b) => a - b);
    }
    cell.value = null;
  } else if (noteMode.value === "corners") {
    if (!cell.cornerNotes.includes(number)) {
      cell.cornerNotes.push(number);
      cell.cornerNotes.sort((a, b) => a - b);
    }
    cell.value = null;
  } else {
    cell.value = number;
    cell.centerNote = [];
    cell.cornerNotes = [];
  }

  // Save to localStorage
  saveToLocalStorage();

  // Check if game is complete
  checkGameComplete();
};

const eraseCell = () => {
  if (!selectedCell.value || isSelectedCellFixed() || isGameComplete.value)
    return;

  const { row, col } = selectedCell.value;
  const cell = gameGrid.value[row][col];

  saveState();

  cell.value = null;
  cell.centerNote = [];
  cell.cornerNotes = [];

  // Save to localStorage
  saveToLocalStorage();
};

const goBack = () => {
  navigateTo("/");
};

const resetGame = () => {
  if (isGameComplete.value) return;

  saveState();

  for (let row = 0; row < 9; row++) {
    for (let col = 0; col < 9; col++) {
      if (!fixedCells.value[row][col]) {
        gameGrid.value[row][col] = {
          value: null,
          centerNote: [],
          cornerNotes: [],
        };
      }
    }
  }
  selectedCell.value = null;

  // Save to localStorage
  saveToLocalStorage();
};

const handleKeydown = e => {
  if (
    selectedCell.value &&
    !isSelectedCellFixed() &&
    e.key >= "1" &&
    e.key <= "9"
  ) {
    inputNumber(parseInt(e.key));
  } else if (
    selectedCell.value &&
    !isSelectedCellFixed() &&
    (e.key === "Backspace" || e.key === "Delete")
  ) {
    eraseCell();
  }
};

const formatTime = seconds => {
  const minutes = Math.floor(seconds / 60);
  const remainingSeconds = seconds % 60;
  return `${minutes.toString().padStart(2, "0")}:${remainingSeconds
    .toString()
    .padStart(2, "0")}`;
};

let timerInterval = null;

onMounted(() => {
  // Try to load from localStorage first
  if (loadFromLocalStorage()) {
    // Game state loaded from localStorage, restart timer if not complete
    if (!isGameComplete.value) {
      timerInterval = setInterval(() => {
        elapsedTime.value = Math.floor((Date.now() - startTime.value) / 1000);
      }, 1000);
    }
    return;
  }

  // If no localStorage, try useState
  const puzzleState = useState("sudokuPuzzle");
  if (puzzleState.value) {
    try {
      const puzzle = puzzleState.value;
      for (let row = 0; row < 9; row++) {
        for (let col = 0; col < 9; col++) {
          gameGrid.value[row][col] = {
            value: puzzle[row][col],
            centerNote: [],
            cornerNotes: [],
          };
          fixedCells.value[row][col] = puzzle[row][col] !== null;
        }
      }
    } catch (e) {
      console.error("Error loading puzzle data:", e);
    }
  } else {
    // Try to load initial puzzle from localStorage
    try {
      const savedPuzzle = localStorage.getItem("sudokuInitialPuzzle");
      if (savedPuzzle) {
        const puzzle = JSON.parse(savedPuzzle);
        for (let row = 0; row < 9; row++) {
          for (let col = 0; col < 9; col++) {
            gameGrid.value[row][col] = {
              value: puzzle[row][col],
              centerNote: [],
              cornerNotes: [],
            };
            fixedCells.value[row][col] = puzzle[row][col] !== null;
          }
        }
      }
    } catch (e) {
      console.error("Error loading puzzle from localStorage:", e);
    }
  }

  // Start timer (only if not loaded from localStorage)
  startTime.value = Date.now();
  timerInterval = setInterval(() => {
    elapsedTime.value = Math.floor((Date.now() - startTime.value) / 1000);
  }, 1000);
});

onUnmounted(() => {
  if (timerInterval) {
    clearInterval(timerInterval);
  }
});
</script>
