<template>
  <div class="Game">
    <div class="wrapper">
      <h1>Simon Says</h1>
        <div class="game-board">
          <div ref="gameBoard" class="simon">
            <ul>
              <li class="tile red" data-tile="1"></li>
              <li class="tile blue" data-tile="2"></li>
              <li class="tile yellow" data-tile="3"></li>
              <li class="tile green" data-tile="4"></li>
            </ul>
          </div>
        </div>
        <div class="game-info">
          <h2>Round: <span :data-round="round">{{round}}</span></h2>
          <button ref="start_btn" data-action="start" >Start</button>
          <p data-action="lose">Sorry, you lost after <span ref="msg_winRounds"></span> rounds!</p>
        </div>
        <div class="game-options">
          <h2>Game Options:</h2>
          <input type="radio" name="mode" value="easy" checked>Easy<br>
          <input type="radio" name="mode" value="medium">Medium<br>
          <input type="radio" name="mode" value="hard">Hard<br>
        </div>
      <div ref="audioCont" data-action="sound"></div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Game',

  data(){
    return {
      sequence: [],
      copy: [],
      round: 0,
      active: true,
      mode: 'medium',
      difficulty: { easy: 1500, medium:1000, hard:400 },
      regFunctions:{ regClick:null, mouseOnBoard:null, mousUpBoard:null },
    }
  },
  mounted(){
    this.init();
  },
  methods:{

    init() {
			this.$refs.start_btn.addEventListener ('click', ()=>{
        this.startGame();
      });

      let modeElement = document.querySelectorAll( 'input[name=mode]' );

      modeElement.forEach((el)=>{

        el.addEventListener('change', (e)=>{
          this.changeMode(e);
        });
      })


    },

    startGame() {

      this.clearScores();

      let lose_msg = document.querySelector( 'p[data-action="lose"]' );
          lose_msg.style.display = 'none';

			this.newRound();
    },

    // add a new color to the sequence and animate it to the user
		newRound() {

      ++this.round;

			this.sequence.push( this.randomNumber() );
			this.copy = [...this.sequence];
			this.animate( this.sequence );
    },

    // the game is controlled primarily through this function, along with checkLose().
		// Since the player can never actually "win", we just listen for clicks as the user
		// plays the sequence and each time, check if they lost
		registerClick(e) {
			let desiredResponse = this.copy.shift();
      let actualResponse  = e.target.getAttribute('data-tile');
          actualResponse  = parseInt(actualResponse, 10);

			this.active = (desiredResponse === actualResponse);
			this.checkLose();
    },

    // three possible situations:
		// 1. The user clicked the wrong color (end the game)
		// 2. The user entered the right color, but is not finished with the sequence (do nothing)
		// 3. The user entered the right color and just completed the sequence (start a new round)
		checkLose() {
			// copy array will be empty when user has successfully completed sequence
			if (this.copy.length === 0 && this.active) {
				this.deactivateSimonBoard();
				this.newRound();

			} else if (!this.active) {
        // user lost
				this.deactivateSimonBoard();
				this.endGame();
			}
    },
    endGame() {
			// notify the user that they lost and change the "round" text to zero
      let lose_msg = document.querySelector( 'p[data-action="lose"]' );
          lose_msg.style.display = 'block';

      this.$refs.msg_winRounds.innerText = this.round
			this.round = 0
		},

    changeMode(e) {
			this.mode = e.target.value;
    },

    /*----------------- Helper functions -------------------*/


    animate(sequence) {
			let i = 0;
      let intervalSpeed = this.difficulty[this.mode];

			let interval = setInterval( ()=> {

				this.playSound(sequence[i]);
				this.lightUp(sequence[i]);

				i++;
				if (i >= sequence.length) {
					clearInterval(interval);
					this.activateSimonBoard();
				}
			}, intervalSpeed );
    },

    // we are embedding the sound file on the fly for the following benefits:
		// 1. ability to play multiple sounds in a row without waiting for the first to complete,
		// 2. <audio> tag provides our fallbacks (ogg, mp3).
		playSound(tile) {

      this.removeAudio();

      let src_mp3     = document.createElement("source");
          src_mp3.src = require(`@/assets/sounds/${tile}.mp3`);
          src_mp3.type= "audio/mp3";

      let ogg     = document.createElement("source");
          ogg.src = require(`@/assets/sounds/${tile}.ogg`);
          ogg.type= "audio/ogg";

      let audio          = document.createElement("audio");
          audio.autoplay = true;
          audio.appendChild( src_mp3 );
          audio.appendChild( ogg );

      this.$refs.audioCont.appendChild(audio);
    },

    lightUp(tileNum) {

      let tile = document.querySelector( `[data-tile='${tileNum}']` );
          tile.classList.add("lit");

      window.setTimeout(function() {
        tile.classList.remove('lit');
      }, 300);
    },

    // allow user to interact with the game
		activateSimonBoard() {
      this.deactivateSimonBoard();
      let gameBoard = this.$refs.gameBoard;

      this.regFunctions.registerClick = (ev)=>{ this.registerClick(ev) };
      this.regFunctions.mouseOnBoard  = (ev)=>{

        ev.target.classList.add("active");
        this.playSound( ev.target.getAttribute('data-tile') );
      };
      this.regFunctions.mousUpBoard   = (ev)=>{ ev.target.classList.remove('active') };

      gameBoard.addEventListener('click', this.regFunctions.registerClick );
      gameBoard.addEventListener('mousedown', this.regFunctions.mouseOnBoard );
      gameBoard.addEventListener('mouseup', this.regFunctions.mousUpBoard );

      gameBoard.children.forEach( (el)=>{
        el.classList.add("hoverable");
      });
    },
    // prevent user from interacting until sequence is done animating
		deactivateSimonBoard() {

      let gameBoard = this.$refs.gameBoard;

      gameBoard.removeEventListener("click", this.regFunctions.registerClick);
      gameBoard.removeEventListener("mousedown", this.regFunctions.mouseOnBoard);
      gameBoard.removeEventListener("mouseup", this.regFunctions.mousUpBoard)


      gameBoard.children.forEach( (el)=>{
        el.classList.remove("hoverable");
      });
		},

    randomNumber() {
			// between 1 and 4
			return Math.floor((Math.random()*4)+1);
    },

    removeAudio(){

      this.$refs.audioCont.children.forEach((el)=>{el.remove()})
    },
    clearScores(){

      this.sequence = [];
			this.copy = [];
      this.round = 0;
      this.active = true;
    }
  },
}
</script>


