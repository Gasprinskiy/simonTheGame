<template>
  <div id="app">
    <div class="text-h2">Simon says</div>
    <div class="game-board-container">
      <div class="game-board-wrapper">
        <div class="game-board" v-if="!isGameDisabled">
          <button
          class="board-btns"
          :class="{clicked: block.isClicked}" 
          v-for="block in simonBlocks"
          :id="`btn` + block.num" 
          :key="block.num"  
          ref="block"
          @mousedown="block.isClicked = true"
          @mouseup="block.isClicked = false"
          @click="checkUserInput(block.num)"/>
        </div>
        <div class="game-board" v-else>
          <button
          class="board-btns"
          v-for="block in simonBlocks"
          :id="`btn` + block.num" 
          :key="block.num"
          ref="block"
          />
        </div>
        <button v-if="counter === 0" class="start-btn" @click="startRound">начать</button>
      </div>
      
      <div class="game-board-settings">
        <div class="text-h3">Раунд: {{counter}}</div>
        <div v-if="isUserLose" class="text-lose-msg">
          Вы проиграли спустя {{scoreCounter}} {{wordDepenseNumber}} попробуйте еще раз
        </div>
        <div class="game-options">
          <div class="text-h3">Cложность:</div>
          <div 
          class="optionons-radio-btn"
          v-for="option in gameOptions" 
          :key="option.value"
          >
            <input 
              type="radio" 
              name="game-mode"
              @input="updateDifficulty"
              :value="option.value"
              :checked="option.isCheked"
            >
            <span>{{option.label}}</span>
          </div>
          
        </div>
      </div>
    </div>
    
    <div ref="sound"></div>
  </div>

</template>

<script>

export default {
  name: 'App',
  data(){
    return {
      simonBlocks: [
        {num: 1, isClicked: false},
        {num: 2, isClicked: false},
        {num: 3, isClicked: false},
        {num: 4, isClicked: false},
      ],
      gameOptions: [
        {label: 'Легкий', value: 1500, isCheked: true},
        {label: 'Средний', value: 1000, isCheked: false},
        {label: 'Сложный', value: 400, isCheked: false},
      ],
      activeBlocks: [],
      copy: [],
      counter: 0,
      scoreCounter: 0, 
      interval: 1500,
      isGameDisabled: true,
      isUserLose: false
    }
  },

  computed: {
    wordDepenseNumber(){
      if(this.scoreCounter === 1){
          return 'раунд'
      } else if(this.scoreCounter > 1 && this.scoreCounter < 5){
          return  'раунда'
      } else if(this.scoreCounter >= 5){
          return  'раундов'
      } 
    }
  },
  methods: {
    
    resetGame(){
      this.counter = 0;
      this.activeBlocks = [];
      this.copy = [];
      this.isGameDisabled = true;
    },

    startRound(){
      this.isUserLose = false;
      this.counter += 1;
      this.scoreCounter = 0;
      this.activeBlocks.push(this.randomNum());
      this.copy = this.activeBlocks.slice(0);
      this.showHints(this.activeBlocks);
    },

    checkUserInput(i) {
      const shift = this.copy.shift();
      this.playSound(i)
      if(shift === i && this.copy.length === 0){
        this.isGameDisabled = true;
        this.startRound()
      } else if(shift !== i) {
        this.isUserLose = true;
        this.scoreCounter = this.counter;
        this.resetGame();
      }
    },

    showHints(frequency){
      let i = 0;
			const interval = setInterval(()=> {
				this.playSound(frequency[i])
        this.lightupBlocks(frequency[i])

				i++

				if (i >= frequency.length) {
					clearInterval(interval);
          this.isGameDisabled = false;
				}
			}, this.interval);
    },

    playSound(n){
      const mpegPath = require(`@/assets/sounds/${n}.mp3`);
      const oggPath = require(`@/assets/sounds/${n}.ogg`);
      this.$refs.sound.innerHTML = `<audio autoplay>
          <source src="${mpegPath}" type="audio/mpeg">
          <source src="${oggPath}" type="audio/ogg">
        </audio>`
    },

    lightupBlocks(n){
      const val = n - 1;
      this.$refs.block[val].classList.toggle('lighted');
      window.setTimeout(()=>{
        this.$refs.block[val].classList.remove('lighted')
      }, this.interval - 100)
    },

    updateDifficulty(e){
      this.interval = Number(e.target.value)
    },

    randomNum() {
      return Math.floor((Math.random() * 4) + 1)
    },
  },
  
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.game-board-container {
  width: 100%;
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin: auto;
}

.text-h2 {
  margin: 0px 0px 50px 0px;
  font-size: 30px;
  font-weight: 600;
}

.text-h3 {
  font-size: 25px;
  margin-bottom: 15px;
}

.game-board-wrapper {
  width: 49%;
  display: flex;
  justify-content: flex-end;
  position: relative;
}

.game-board {
  min-width: 350px;
  height: 350px;
  border-radius: 50%;
  display: flex;
  flex-wrap: wrap;
  overflow: hidden;
  justify-content: center;
}

.board-btns {
  min-width: 50%;
  border: none;
  outline: none;
  cursor: pointer;
}

#btn1 {
  background: #77BCFF;
}
#btn2 {
  background: #FF998D;
}
#btn3 {
  background: #FEF584;
}
#btn4 {
  background: #D7EB72;
}

.board-btns.lighted,
.board-btns.clicked {
  opacity: 0.5;
}



.start-btn {
  width: 90px;
  height: 90px;
  position: absolute;
  border-radius: 50%;
  top: 135px;
  right: 130px;
  border: none;
  cursor: pointer;
  z-index: 1000;
  background-color: rgba(245, 245, 245, 0.342);
  backdrop-filter: blur(3px);
  color: rgb(0, 0, 0);
  text-transform: uppercase;
  font-size: 18px;
  font-weight: 500;
}

.start-btn:hover {
  backdrop-filter: blur(10px);
}

.game-board-settings {
  width: 46%;
  margin-top: 30px;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  text-align: left;
}

.text-lose-msg {
  text-align: left;
  width: 40%;
  line-height: 20px;
  margin-bottom: 15px;
}

.optionons-radio-btn {
  margin-bottom: 5px;
  font-size: 18px;
}

.optionons-radio-btn span {
  margin-left: 5px;
}




</style>
