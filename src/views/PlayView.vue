<template>
  <header>
    <div>
      <Modal v-bind:uiLabels="uiLabels" v-bind:lang="lang" v-bind:sourceName="sourceName"
        v-on:switchLanguage="switchLanguage">
        <button v-on:click="togglePopup"></button>
      </Modal>
    </div>
  </header>

  <div class="direction">
    <br>
    {{ uiLabels.selectedDirection }} <br>
    <button v-if="this.inputDirection == 'Horizontal'" v-on:click="changeDirectionToH" id="arrowbutton">
      →
    </button>
    <button v-else style="opacity: 50%" v-on:click="changeDirectionToH" id="arrowbutton">
      →
    </button>
    <button v-if="this.inputDirection == 'Vertical'" v-on:click="changeDirectionToV" id="arrowbutton">
      ↓
    </button>
    <button v-else style="opacity: 50%" v-on:click="changeDirectionToV" id="arrowbutton">
      ↓
    </button>
  </div>


  <div id="div2">
    <Crossword v-on:sendPosition="this.storePosition($event)" v-on:updateLayout="this.updateLayout($event)"
      v-bind:sourceName="this.sourceName" v-bind:crossword="this.userCrossword" v-bind:cellsAmount="this.cellsAmount">
    </Crossword>
  </div>

  <div class="wordDescriptionWrapper">

    <div>
      <button id="finishedGame" @click="$router.push('/lobby/' + lang)">{{ uiLabels.finishedGame }}</button>
    </div>
    <div id="horizontalDescriptions">

      <div id="wordDescTop">{{ uiLabels.horizontalWords }}</div>
      <div id="orderedList" class="playScroll">
        <ul style="list-style: none;">
          <li v-for="(value, key) in this.getSortedDescs()" :key="key">
            <span v-if="value.direction == 'Horizontal'">
              {{ value.wordInOrder + ". " + value.desc }}
            </span>
            <span v-else style="display: none;">
            </span>
          </li>
        </ul>
      </div>

    </div>



    <div id="verticalDescriptions">

      <div id="wordDescTop">{{ uiLabels.verticalWords }}</div>
      <div id="orderedList" class="playScroll">
        <ul style="list-style: none;">
          <li v-for="(value, key) in this.getSortedDescs()" :key="key">
            <span v-if="value.direction == 'Vertical'">
              {{ value.wordInOrder + ". " + value.desc }}
            </span>
            <span v-else style="visibility: hidden;">
            </span>
          </li>
        </ul>
      </div>
    </div>


  </div>

</template>


<script>

import Crossword from '../components/Crossword.vue'
import Modal from '../components/PopUp.vue'
import io from 'socket.io-client';

const socket = io();

export default {
  name: 'PlayView',
  components: {
    Crossword,
    Modal
  },
  props: {
    modal: Object,
  },
  data: function () {
    return {
      receivedCross: null,

      word: "",
      desc: "",
      cellsAmount: 8,
      occupiedPosition: { x: null, y: null },
      latestOccupied: { x: 0, y: 0 },
      crosswordAnswer: [],
      userCrossword: [],

      crosswordPackage: {},

      firstMouseClick: true,

      showModal: false,
      uiLabels: {},
      gameID: "",
      lang: "",
      sourceName: "PlayView",
      inputDirection: "Horizontal",
      arrow: "&rarr;"
    }
  },

  created:
    function () {
      socket.emit('pageLoaded')
      socket.on("init", (labels) => {
        this.uiLabels = labels
      })

      this.gameID = this.$route.params.gameID;

      socket.on("dataUpdate", (data) =>
        this.data = data
      ),

        socket.emit("joinPoll", this.gameID);


      socket.on('gameToBePlayed', data => {
        this.receivedCross = JSON.parse(JSON.stringify(data))

        this.loadReceivedCrossword();
        this.userCrossword = this.getUserCrossword()

        this.gameID = JSON.parse(JSON.stringify(this.receivedCross.crosswordName))
        this.cellsAmount = JSON.parse(JSON.stringify(this.receivedCross.cellsAmount))
        this.wordDesc = JSON.parse(JSON.stringify(this.receivedCross.wordDesc))
      })

      socket.on('sendUpdatedUserCross', (d) => {
        
        if (d.gameID == this.gameID) {
          this.userCrossword = d.crossword
        }
      })

      window.addEventListener('keydown', this.enterLetterFromKeyPress)
    },

  beforeUnmount() {
    window.removeEventListener('keydown', this.enterLetterFromKeyPress)
  },

  methods: {
    storePosition: function (event) {

      if (this.firstMouseClick) {
        this.occupiedPosition.x = event.x
        this.occupiedPosition.y = event.y
        this.userCrossword[this.occupiedPosition.y][this.occupiedPosition.x].isOccupied = true
      } else {
        this.userCrossword[this.occupiedPosition.y][this.occupiedPosition.x].isOccupied = false
        this.userCrossword[event.y][event.x].isOccupied = true
      }

      this.firstMouseClick = false

      this.occupiedPosition.x = event.x
      this.occupiedPosition.y = event.y

      socket.emit('updatedUserCrossword', { crossword: this.userCrossword, gameID: this.gameID })


    },

    enterLetterFromKeyPress: function (event) {



      if ((event.which >= 65 && event.which <= 90) || (event.which >= 97 && event.which <= 122) ||
        event.which == 192 || event.which == 221 || event.which == 222) {

        if (this.inputDirection == "Horizontal" && typeof (event.key) == "string") {

          this.userCrossword[this.occupiedPosition.y][this.occupiedPosition.x].letter = event.key.toUpperCase()

          if (this.crosswordAnswer[this.occupiedPosition.y][this.occupiedPosition.x + 1].letter != null ||
            this.crosswordAnswer[this.occupiedPosition.y][this.occupiedPosition.x + 1].letter != undefined) {

            this.userCrossword[this.occupiedPosition.y][this.occupiedPosition.x].isOccupied = false
            this.occupiedPosition.x++

            this.userCrossword[this.occupiedPosition.y][this.occupiedPosition.x].isOccupied = true
            this.firstMouseClick = false
          }
        } else {
          this.userCrossword[this.occupiedPosition.y][this.occupiedPosition.x].letter = event.key.toUpperCase()

          if (this.crosswordAnswer[this.occupiedPosition.y + 1][this.occupiedPosition.x].letter != null ||
            this.crosswordAnswer[this.occupiedPosition.y + 1][this.occupiedPosition.x].letter != undefined) {

            this.userCrossword[this.occupiedPosition.y][this.occupiedPosition.x].isOccupied = false
            this.occupiedPosition.y++

            this.userCrossword[this.occupiedPosition.y][this.occupiedPosition.x].isOccupied = true
            this.firstMouseClick = false
          }
        }
      } else if (event.key == "Backspace") {
        if (this.inputDirection == "Horizontal") {
          if ((this.userCrossword[this.occupiedPosition.y][this.occupiedPosition.x - 1].letter != null ||
            this.userCrossword[this.occupiedPosition.y][this.occupiedPosition.x - 1].letter != undefined) &&
            this.userCrossword[this.occupiedPosition.y][this.occupiedPosition.x].letter == null) {

            this.userCrossword[this.occupiedPosition.y][this.occupiedPosition.x - 1].letter = null

            this.userCrossword[this.occupiedPosition.y][this.occupiedPosition.x].isOccupied = false
            this.occupiedPosition.x--
            this.userCrossword[this.occupiedPosition.y][this.occupiedPosition.x].isOccupied = true

          } else if (this.crosswordAnswer[this.occupiedPosition.y][this.occupiedPosition.x - 1].letter == null ||
            this.crosswordAnswer[this.occupiedPosition.y][this.occupiedPosition.x - 1].letter == undefined) {

            this.userCrossword[this.occupiedPosition.y][this.occupiedPosition.x].letter = null

          } else if (this.crosswordAnswer[this.occupiedPosition.y][this.occupiedPosition.x].letter != null ||
            this.crosswordAnswer[this.occupiedPosition.y][this.occupiedPosition.x + 1].letter != null ||
            this.crosswordAnswer[this.occupiedPosition.y][this.occupiedPosition.x + 1].letter != undefined) {

            this.userCrossword[this.occupiedPosition.y][this.occupiedPosition.x].letter = null

          }
        } else {
          if ((this.userCrossword[this.occupiedPosition.y - 1][this.occupiedPosition.x].letter != null ||
            this.userCrossword[this.occupiedPosition.y - 1][this.occupiedPosition.x].letter != undefined) &&
            this.userCrossword[this.occupiedPosition.y][this.occupiedPosition.x].letter == null) {

            this.userCrossword[this.occupiedPosition.y - 1][this.occupiedPosition.x].letter = null

            this.userCrossword[this.occupiedPosition.y][this.occupiedPosition.x].isOccupied = false
            this.occupiedPosition.y--
            this.userCrossword[this.occupiedPosition.y][this.occupiedPosition.x].isOccupied = true

          } else if (this.crosswordAnswer[this.occupiedPosition.y - 1][this.occupiedPosition.x].letter == null ||
            this.crosswordAnswer[this.occupiedPosition.y - 1][this.occupiedPosition.x].letter == undefined) {

            this.userCrossword[this.occupiedPosition.y][this.occupiedPosition.x].letter = null

          } else if (this.crosswordAnswer[this.occupiedPosition.y][this.occupiedPosition.x].letter != null ||
            this.crosswordAnswer[this.occupiedPosition.y + 1][this.occupiedPosition.x].letter != null ||
            this.crosswordAnswer[this.occupiedPosition.y + 1][this.occupiedPosition.x].letter != undefined) {

            this.userCrossword[this.occupiedPosition.y][this.occupiedPosition.x].letter = null
          }
        }
      }

      let allMatchesCorrect = true
      for (let v = 0; v < this.cellsAmount; v++) {
        for (let h = 0; h < this.cellsAmount; h++) {
          if (typeof (this.crosswordAnswer[v][h].letter) == "string") {
            this.crosswordAnswer[v][h].letter = JSON.parse(JSON.stringify(this.crosswordAnswer[v][h].letter.toUpperCase()))
          }

          if (this.crosswordAnswer[v][h].letter != this.userCrossword[v][h].letter) {
            allMatchesCorrect = false
          }
        }
      }


      socket.emit('updatedUserCrossword', { crossword: this.userCrossword, gameID: this.gameID })



      if (allMatchesCorrect) {
        alert(this.uiLabels.youMadeIt)

      }
    },

    getUserCrossword: function () {

      let tempUserCrossword = JSON.parse(JSON.stringify(this.crosswordAnswer))

      tempUserCrossword.forEach((outerList) => {
        outerList.forEach(element => {
          element.letter = null
        })
      })

      return tempUserCrossword
    },

    loadReceivedCrossword: function () {

      this.crosswordAnswer = JSON.parse(JSON.stringify(this.receivedCross.crossword))

    },
    
    getSortedDescs: function () {
      return this.receivedCross.wordDesc.sort((a, b) => a.wordInOrder - b.wordInOrder)
    },

    changeDirectionToV: function () {
      if (this.inputDirection === "Horizontal") {
        this.inputDirection = "Vertical"
      }
    },

    changeDirectionToH: function () {
      if (this.inputDirection === "Vertical") {
        this.inputDirection = "Horizontal"
      }
    },

    sendCrosswordPackage: function () {
      this.crosswordPackage.crosswordName = this.gameID
      this.crosswordPackage.crossword = this.userCrossword
      this.crosswordPackage.cellsAmount = this.cellsAmount
      socket.emit("createdCrosswordPackage", this.crosswordPackage)
    },

    switchLanguage: function () {
      if (this.lang === "en")
        this.lang = "sv"
      else
        this.lang = "en"

      socket.emit("switchLanguage", this.lang)
      this.$router.push(this.lang)
    },

    togglePopup: function () {
      this.showModal = !this.showModal;
    },

  }
}

</script>

<style>

#arrowbutton {
  font-size: x-large;
  justify-content: center;
  align-items: flex-start;
}

#div2 {
  float: left;
  margin-top: 15vw;
  margin-left: 6vw;
}

.wordDescriptionWrapper {
  display: flex;
  flex-direction: column;
  float: right;
  align-items: center;
  height: 100vh;
  width: 22vw;
  margin-right: 6vw;
}

#horizontalDescriptions {
  width: 20vw;
  height: 31vh;
  border-radius: 5px;
  border-color: #a6d8d4;
  margin-left: 5%;
  margin-top: 6%;
  color: white;
  background-color: #43918a;
  font-family: "Comic Sans MS", "Comic Sans";
  font-size: 2vw;
  position: relative;
  float: left;
  margin-bottom: 2rem;
}

#verticalDescriptions {
  width: 20vw;
  height: 31vh;
  border-radius: 5px;
  border-color: #a6d8d4;
  margin-left: 5%;
  color: white;
  background-color: #43918a;
  font-family: "Comic Sans MS", "Comic Sans";
  font-size: 2vw;
  position: relative;
  float: left;
}

#wordDescTop {
  width: 19.2vw;
  height: 6vh;
  border-top-left-radius: 0.5vw;
  border-top-right-radius: 0.5vw;
  border-color: #43918a;
  margin-left: 5%;
  color: #43918a;
  background-color: white;
  font-family: "Comic Sans MS", "Comic Sans";
  font-size: 2.2vw;
  position: relative;
  float: left;
  margin: 0.4vw;
  text-align: center;
}

#orderedList {
  width: 19vw;
  margin-left: 0.4vw;
  height: 22vh;
  overflow-y: auto;
  border: 0.1vw solid white;
  border-bottom-left-radius: 0.5vw;
  border-bottom-right-radius: 0.5vw;
}

ol {
  margin-block-start: 0;
  margin-block-end: 0;
}


#finishedGame {
  width: 10vw;
  height: 3vw;
  background-color: #FE5F55;
  border-radius: 5px;
  color: white;
  font-family: "Comic Sans MS", "Comic Sans";
}

button {
  width: 7.5vw;
  height: 5vh;
  border-radius: 1vw;
  border-color: #ba0c00;
  margin-top: 2.5vw;
  margin-left: 0.8vw;
  margin-right: 0.8vw;
  margin-bottom: 3vw;
  color: white;
  background-color: #FE5F55;
  font-family: "Comic Sans MS", "Comic Sans";
  font-size: 1vw;
  cursor: pointer;
  position: relative;
}

div.playScroll {
  overflow-x: hidden;
  overflow-y: auto;
  text-align: justify;
  font-size: 1.5vw;
}

.playScroll::-webkit-scrollbar {
  width: 2vh;
  border-radius: 2vw;
}

.playScroll::-webkit-scrollbar-track {
  -webkit-box-shadow: inset 0 0 6px rgba(255, 253, 253, 0.3);
}

.playScroll::-webkit-scrollbar-thumb {
  -webkit-box-shadow: inset 0 0 6px rgba(0, 0, 0, 0.5);
  background: #43918a;
  border-bottom-right-radius: 0.5vw;
  border-bottom-left-radius: 0.5vw;
}

.direction {
  width: 20vw;
  height: 10vw;
  position: absolute;
  border-style: solid;
  background-color: #93b39c;
  border-color: #43918a;
  margin-left: 15vw;
  margin-top: 2vw;
  border-radius: 1.5vw;
  justify-content: center;
  font-size: 1.5vw;
}
</style>