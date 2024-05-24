<template>
    <div class="app-container">
      <div class="phone-screen">
        <div class="game" @mousemove="updateCursor">
          <!-- Display the congratulatory image when the game is won -->
          <div v-if="showCongrats" class="congrats-message" @click="goToPlayStore">
            <img src="@/assets/congrats-image.png" class="congrats-image"/>
          </div>
          <div class="grid">
            <!-- Loop through gridLayout to display each cell -->
            <div
              v-for="(cell, index) in gridLayout"
              :key="index"
              class="grid-item"
              :class="{ 'correct': cell.isCorrect, 'empty': !cell.letter }"
            >
              <img v-if="cell.letter" src="@/assets/rect.png" class="grid-background"/>
              <span class="letter" :class="{ 'correct': cell.isCorrect }">{{ cell.letter }}</span>
            </div>
          </div>
          <div class="circle-container">
            <svg class="lines">
              <!-- Draw lines connecting selected letters -->
              <line
                v-for="(line, index) in lines"
                :key="index"
                :x1="line.x1"
                :y1="line.y1"
                :x2="line.x2"
                :y2="line.y2"
                stroke="orange"
                stroke-width="10"
              />
            </svg>
            <img src="@/assets/circle.png" class="circle-background"/>
            <div class="circle" @mousedown="startSelecting" @mousemove="selectLetter" @mouseup="finishSelecting">
              <!-- Loop through wheelLetters to display each letter -->
              <div
                v-for="(letter, index) in wheelLetters"
                :key="index"
                class="wheel-item"
                :class="{'selected': selectedWheelIndices.includes(index)}"
                :data-index="index"
                :data-x="wheelPositions[index].x"
                :data-y="wheelPositions[index].y"
              >
                {{ letter }}
              </div>
              <!-- Shuffle button -->
              <button @click="shuffleLetters" class="shuffle-button">
                <img src="@/assets/suffle.png" alt="Shuffle" class="shuffle-icon">
              </button>
            </div>
          </div>
          <!-- Custom cursor image -->
          <img :src="cursorImage" :style="cursorStyle" class="custom-cursor" />
          <!-- Play now button -->
          <button class="play-now-button" @click="goToPlayStore">PLAY NOW!</button>
        </div>
        <div class="selected-letters-box">
          <!-- Display selected letters -->
          <span v-for="letter in selectedLetters" :key="letter" class="selected-letter">{{ letter }}</span>
        </div>
      </div>
      <!-- Reset button -->
      <button class="reset-button" @click="resetGame">RESET</button>
      <!-- Tutorial message -->
      <div v-if="showTutorial" class="tutorial">
        <div class="tutorial-message">Connect the letters GOLD</div>
      </div>
    </div>
  </template>
  
  <script>
  export default {
    data() {
      return {
        gridLayout: [
          { letter: ' ', isCorrect: false }, { letter: ' ', isCorrect: false }, { letter: ' ', isCorrect: false }, { letter: ' ', isCorrect: false },
          { letter: ' ', isCorrect: false }, { letter: '', isCorrect: false }, { letter: ' ', isCorrect: false }, { letter: '', isCorrect: false },
          { letter: ' ', isCorrect: false }, { letter: ' ', isCorrect: false }, { letter: ' ', isCorrect: false }, { letter: '', isCorrect: false },
          { letter: '', isCorrect: false }, { letter: '', isCorrect: false }, { letter: '', isCorrect: false }, { letter: '', isCorrect: false }
        ],
        wheelLetters: ['L', 'D', 'G', 'O'],
        correctWords: {
          gold: [0, 1, 2, 3],
          god: [0, 4, 8],
          log: [2, 6, 10],
          dog: [8, 9, 10]
        },
        selectedLetters: '',
        selectedWheelIndices: [],
        isSelecting: false,
        showCongrats: false,
        lines: [],
        wheelPositions: [
          { x: 155, y: 5 }, // Position for L
          { x: 90, y: 80 }, // Position for D
          { x: 220, y: 80 }, // Position for G
          { x: 150, y: 140 }  // Position for O
        ],
        showTutorial: false
      };
    },
    mounted() {
      this.startTutorial();
    },
    methods: {
      startTutorial() {
        setTimeout(() => {
          this.showTutorial = true;
          this.animateHand();
        }, 5000);
      },
      animateHand() {
        const positions = [2, 3, 0, 1]; // Corresponding positions for 'G', 'O', 'L', 'D'
        let delay = 0;
  
        positions.forEach((pos, index) => {
          const { x, y } = this.wheelPositions[pos];
          setTimeout(() => {
            this.handPosition = {
              top: `${y}px`,
              left: `${x}px`
            };
            if (index > 0) {
              const previousPos = this.wheelPositions[positions[index - 1]];
              this.lines.push({
                x1: previousPos.x,
                y1: previousPos.y,
                x2: x,
                y2: y
              });
            }
            this.selectedWheelIndices.push(pos);
          }, delay);
          delay += 1000; // Move the hand every second
        });
  
        setTimeout(() => {
          this.showTutorial = false;
          this.resetHandPosition();
        }, delay + 1000);
      },
      resetHandPosition() {
        this.handPosition = { top: '0px', left: '0px' };
        this.lines = [];
        this.selectedWheelIndices = [];
      },
      startSelecting(event) {
        this.isSelecting = true;
        this.selectedLetters = '';
        this.selectedWheelIndices = [];
        this.lines = [];
        const index = parseInt(event.target.dataset.index);
        if (!isNaN(index)) {
          this.selectedWheelIndices.push(index);
          this.selectedLetters += this.wheelLetters[index];
        }
      },
      selectLetter(event) {
        if (this.isSelecting) {
          const index = parseInt(event.target.dataset.index);
          if (!this.selectedWheelIndices.includes(index) && !isNaN(index)) {
            const previousIndex = this.selectedWheelIndices[this.selectedWheelIndices.length - 1];
            const previousPosition = this.wheelPositions[previousIndex];
            const currentPosition = this.wheelPositions[index];
            this.lines.push({
              x1: previousPosition.x,
              y1: previousPosition.y,
              x2: currentPosition.x,
              y2: currentPosition.y
            });
            this.selectedWheelIndices.push(index);
            this.selectedLetters += this.wheelLetters[index];
          }
        }
      },
      finishSelecting() {
        this.isSelecting = false;
        if (this.checkWord()) {
          this.placeLetters();
          if (this.allWordsFound()) {
            this.showCongrats = true;
          }
        }
        this.selectedLetters = '';
        this.selectedWheelIndices = [];
        this.lines = [];
      },
      checkWord() {
        return Object.prototype.hasOwnProperty.call(this.correctWords, this.selectedLetters.toLowerCase());
      },
      placeLetters() {
        const positions = this.correctWords[this.selectedLetters.toLowerCase()];
        positions.forEach((pos, index) => {
          this.gridLayout[pos].letter = this.selectedLetters[index].toUpperCase();
          this.gridLayout[pos].isCorrect = true;
        });
      },
      allWordsFound() {
        const words = ['GOLD', 'GOD', 'LOG', 'DOG'];
        return words.every(word => {
          const positions = this.correctWords[word.toLowerCase()];
          return positions.every(pos => this.gridLayout[pos].letter === word[positions.indexOf(pos)]);
        });
      },
      resetGame() {
        this.gridLayout = [
          { letter: ' ', isCorrect: false }, { letter: ' ', isCorrect: false }, { letter: ' ', isCorrect: false }, { letter: ' ', isCorrect: false },
          { letter: ' ', isCorrect: false }, { letter: '', isCorrect: false }, { letter: ' ', isCorrect: false }, { letter: '', isCorrect: false },
          { letter: ' ', isCorrect: false }, { letter: ' ', isCorrect: false }, { letter: ' ', isCorrect: false }, { letter: '', isCorrect: false },
          { letter: '', isCorrect: false }, { letter: '', isCorrect: false }, { letter: '', isCorrect: false }, { letter: '', isCorrect: false }
        ];
        this.selectedLetters = '';
        this.selectedWheelIndices = [];
        this.showCongrats = false;
        this.lines = [];
      },
      shuffleLetters() {
        this.wheelLetters = this.wheelLetters
          .map(letter => ({ letter, sort: Math.random() }))
          .sort((a, b) => a.sort - b.sort)
          .map(({ letter }) => letter);
      },
      goToPlayStore() {
        window.location.href = "https://play.google.com/store/apps/details?id=com.fugo.wow&hl=tr&pli=1";
      }
    }
  };
  </script>
  
  <style scoped src="../assets/kelimebulmaca.css"></style>
  