<template>

	<div class="title">

		<div class="title__logo message">
			<h1>~PipQuest~</h1>
			<p>A Retro Adventure<br>by <a href="http://www.gregorterrill.com" target="_blank">Gregor Terrill</a></p>
		</div>

		<div class="title__menu message">

			<ul class="title__options">
				<li class="title__option"><a @click.prevent="startNewGame()" :class="'title__link' + ((currentOption === 0) ? ' active': '')" href="#">NEW ADVENTURE</a></li>
				<li v-if="this.savedLevel" class="title__option"><a @click.prevent="continueGame()" :class="'title__link' + ((currentOption === 1) ? ' active': '')" href="#">CONTINUE</a></li>
			</ul>

			<div class="title__savedata" v-if="this.savedLevel">
				<p>WORLD:&nbsp;{{ savedLevel }}</p>
				<p>XP:&nbsp;&nbsp;&nbsp;&nbsp;{{ savedXP }}</p>
			</div>

		</div>

  	<audio ref="sound-blip" :src="getAudioSource('blip')" preload />
  </div>
</template>

<script>
import store from '../data/store.js'

export default {
	props: [],
	data() {
  	return {
  		currentOption: 0,
  		savedLevel: null,
  		savedXP: null,
  		keystrokes: []
	  }
  },
  methods: {

  	getSaveData(itemName) {
  		if (localStorage.getItem(itemName) !== null) {
	      return parseInt(localStorage.getItem(itemName));
	    } else {
	    	return null;
	    }
  	},
		
		getAudioSource(soundName) {
  		return require('../assets/sound/' + soundName + '.wav');
  	},

		playBlip() {
			const soundEl = this.$refs['sound-blip'];

			if (soundEl) {
				soundEl.volume = 0.15;
				soundEl.play();
				soundEl.currentTime = 0;
			}
		},

		setOption(optionNum) {
			this.playBlip();
			this.currentOption = optionNum;
		},

		checkForKCode(keyCode) {

			const kCode = [38,38,40,40,37,39,37,39,66,65],
						position = this.keystrokes.length - 1;

			if (position === -1 || this.keystrokes[position] === kCode[position] ) {
				this.keystrokes.push(keyCode);
				
				if (this.keystrokes.length === kCode.length) {
					//activate cheats
					store.windows.menu.cheats = true;
					this.keystrokes = [];

					//tripblip
					this.playBlip();
					setTimeout(() => {
						this.playBlip()
					}, 200);
					setTimeout(() => {
						this.playBlip()
					}, 400);
				}

			} else {
				this.keystrokes = [];
			}
		},

  	handleKeyPress(e) {

  		if (!store.windows.menu.cheats) {
  			this.checkForKCode(e.keyCode);
  		}

			switch(e.keyCode) {

				case 13: //enter
				case 32: //space
					if (this.currentOption === 0) {
						this.startNewGame();
					} else {
						this.continueGame();
					}
					break;

				case 87: //w
		    case 38: //up
		    	if (this.currentOption !== 0) {
		    		this.playBlip();
		    	}
					this.currentOption = 0;
					break;

				case 83: //s
		    case 40: //down
		    	if (this.savedLevel === null) {
		    		return false;
		    	}
		    	if (this.currentOption !== 1) {
		    		this.playBlip();
		    	}
					this.currentOption = 1;
		      break;
			}
		},

		startNewGame() {
			localStorage.removeItem('currentLevel');
			localStorage.removeItem('playerXP');
			store.currentLevelNum = 1;
			store.player.xp = 0;
			store.currentLevel = JSON.parse(JSON.stringify(store.levels[store.currentLevelNum]));
			store.windows.title.open = false;

			ga('send', 'event', 'PipQuest', 'newGame');

		},

		continueGame() {
			store.windows.title.open = false;

			ga('send', 'event', 'PipQuest', 'continueGame');
		}

  },
  mounted() {
  	this.savedLevel = this.getSaveData('currentLevel');
  	this.savedXP = this.getSaveData('playerXP');
  	window.addEventListener('keyup', this.handleKeyPress);
  	this.playBlip();
  },
  beforeDestroy() {
  	window.removeEventListener('keyup', this.handleKeyPress);
  	this.playBlip();
  }
}
</script>

<style lang="scss">
.title__logo {
	z-index:101;
	position:fixed;
	width:45rem;
	top:20vh;
  left:50%;
  margin-left:-22.5rem;
  text-align:center;

	h1 {
		color:#FFF;
		text-align:center;
		font-size:64px;
		margin:2rem 1rem;
	}

	p {
		display:block;
		text-align:center;
		margin-top:2rem;
	}
}
.title__menu {
  z-index:101;
  position:fixed;
  width:20rem;
  bottom:20vh;
  left:50%;
  margin-left:-10rem;
}
.title__options {
	list-style-type:none;
	padding:0;
	margin:0;
}
.title__link {
	text-align:center;
	color:white;
	border:2px solid transparent;
	display:block;
	padding:1rem;

	&.active, &:hover { border-color:white; color: white; }
	&:focus, &:active { color:white; }
}
.title__savedata {
	margin-top:1rem;
	padding-left:4.5rem;
}
</style>