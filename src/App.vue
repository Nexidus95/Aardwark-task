<template>
 <div style="mains">
 <h1>Aardvrak praktikinė užduotis</h1>
  <div>
    <h4>Api base URL</h4>
    <input v-model="link" @change="GetConfiguration()" placeholder="API address" />
  </div>
  <div style="padding: 50px 0px;">
   <h4>Last 200 games stats</h4>
   <table class = "stats">
    <tbody>
     <tr>
       <td></td>
       <th class = "cell-cold" colspan = "5">Cold</th>
       <th :colspan = "this.configurations.slots-10">Neutral</th>
       <th class = "cell-hot" colspan = "5">Hot</th>
     </tr>
      <th>Slot</th>
      <td class = "slot" v-for="number in mostwon" :key="number.result" v-bind:style="{ backgroundColor: configurations.colors[number.result]}">{{configurations.results[number.result]}}</td>
     <tr>
      <th>Hits</th>
      <td v-for="(number,index) in mostwon" :key="index">{{number.count}}</td>
     </tr>
    </tbody>
   </table>
  </div>
  <div>
  <div class = "gameboard">
   <h4>Gameboard</h4>
   <div class = "board">
    <button v-for="config in configurations.positionToId" :key="config.id" v-bind:style="{ backgroundColor: configurations.colors[config]}">{{configurations.results[config]}}</button>
   </div>
   <div v-if= "configurations.slots == 38" class="spinner">
     <img v-if="spinning" class="wheelSpin" src="./components/American.png">
     <img v-if="!spinning" class="wheelStill" src="./components/American.png">
   <div class="w3-display-middle w3-large" v-if="winning">Result: {{previous.result}}</div>
    </div>
    <div v-else-if= "configurations.slots == 37" class="spinner">
     <img v-if="spinning" class="wheelSpin" src="./components/Europe.png">
     <img v-if="!spinning" class="wheelStill" src="./components/Europe.png">
     <div class="w3-display-middle w3-large" v-if="winning">Result: {{previous.result}}</div>
    </div>
  </div>
  <div class="eventDiv">
    <h4>Events</h4>
      <table class="eventTable" :key="componentKey">
        <tbody>
          <tr class="eventTr" v-for="playedgames in events" v-if="playedgames" :key="playedgames.id">{{playedgames}}</tr>
        </tbody>
      </table>
    </div>
    <div class="log">
      <span>LOG</span>
        <p style="white-space: pre-line; backgroundColor:Gainsboro; ba">{{ message }}</p>
       <br>
      </div>
  </div>
 </div>
</template>

<script>
export default {
  data () {
    return {
      message: '',
      mostwon: {},
      configurations: {},
      interval: null,
      link: 'https://dev-games-backend.advbet.com/v1/ab-roulette/1',
      nextgame: [],
      currentid: 555,
      previous: [],
      fakecounter: null,
      timecounter: 0,
      spinning: false,
      events: [],
      eventcounter: 0,
      intervals: null,
      nextgames: null,
      componentKey: 0,
      intervalas: null,
      winning: true
    }
  },
  methods: {
    forceRerender () {
      this.componentKey += 1
    },
    MostWon () {
	  clearInterval(this.intervalas)
      this.message = this.message + this.timestamp() + ' GET ../stats?limit=200\n'
      fetch(this.link + '/stats?limit=200')
        .then(response => response.json())
        .then((data) => {
          this.mostwon = data
        })
        .catch(() => {
          this.message = this.message + this.timestamp() + ' GET ../stats?limit=200 failed\n'
          setTimeout(this.message = this.message + this.timestamp() + ' GET ../stats?limit=200 trying again\n', 1000)
          setTimeout(() => this.MostWon(), 3000)
        })
    },
    GetConfiguration () {
	  clearInterval(this.nextgames)
	  clearInterval(this.intervalas)
      if (this.spinning) {
        this.spinningChange()
      }
      fetch(this.link + '/configuration')
        .then(response => response.json())
        .then((data) => {
          this.configurations = data
          this.message = this.message + this.timestamp() + ' GET ../configuration\n'
          this.MostWon()
          setTimeout(() => this.GetNextgame(), 2000)
        })
        .catch(() => {
          this.message = this.message + this.timestamp() + ' GET ../configuration failed\n'
          setTimeout(() => this.ConfigurationErrorText(), 1000)
          setTimeout(() => this.GetConfiguration(), 3000)
        })
    },
    ConfigurationErrorText () {
      this.message = this.message + this.timestamp() + ' GET ../configuration trying again\n'
    },
    GetNextgame () {
      clearInterval(this.nextgames)
	  clearInterval(this.intervalas)
      this.message = this.message + this.timestamp() + ' Checking for new game\n'
      fetch(this.link + '/nextGame')
        .then(response => response.json())
        .then((data) => {
          this.nextgame = data
          this.message = this.message + this.timestamp() + ' GET ../nextGame\n'
          this.previous = this.nextgame.id
          this.fakecounter = this.nextgame.fakeStartDelta
          this.timecounter = this.nextgame.fakeStartDelta
          this.message = this.message + this.timestamp() + ' New round starts in ' + this.fakecounter + ' seconds ' + '\n'
          this.counter()
          setTimeout(() => this.spinningWheelText(), this.fakecounter * 1000)
          setTimeout(() => this.spinningChange(), this.fakecounter * 1000)
          this.nextgames = setTimeout(() => this.GetPrevious(), (this.nextgame.startDelta + 2) * 1000)
        })
        .catch(() => {
          this.message = this.message + this.timestamp() + ' GET ../nextGame failed\n'
          setTimeout(() => this.failedNextGameMessage(), 1000)
          this.nextgames = setTimeout(() => this.GetNextgame(), 3000)
        })
    },
    spinningWheelText () {
      this.message = this.message + this.timestamp() + ' Spinning the wheel\n'
    },
    spinningChange () {
      if (!this.spinning) {
        this.spinning = true
      } else {
        this.spinning = false
      }
    },
    counter () {
      this.intervalas = setInterval(() => {
        if (this.fakecounter === 0) {
          clearInterval(this.intervalas)
        } else {
          this.fakecounter--
          this.events[this.eventcounter] = 'Game ' + this.nextgame.id + ' starts in ' + this.fakecounter
          this.forceRerender()
        }
      }, 1000)
    },
    failedNextGameMessage () {
      this.message = this.message + this.timestamp() + ' GET ../nextGame trying again\n'
    },
    GetPrevious () {
      clearInterval(this.nextgames)
      this.message = this.message + this.timestamp() + ' Getting game result\n'
      fetch(this.link + '/game/' + this.previous)
        .then(response => response.json())
        .then((data) => {
          if (this.nextgame.id != null) {
            this.previous = data
            this.spinningChange()
            this.message = this.message + this.timestamp() + ' Game ' + this.previous.id + ' result: ' + this.previous.result + '\n'
            this.events[this.eventcounter] = 'Game ' + this.previous.id + ' result: ' + this.previous.result
            this.eventcounter++
            this.MostWon()
          }
        })
      this.nextgames = setTimeout(() => this.GetNextgame(), 5000)
    },
    timestamp () {
      const current = new Date()
      const date = current.getFullYear() + '-' + (current.getMonth() + 1) + '-' + current.getDate() + 'T'
      const time = current.getHours() + ':' + current.getMinutes() + ':' + current.getSeconds() + '.' + current.getMilliseconds() + 'Z'
      const dateTime = date + time
      return dateTime
    }
  },
  created () {
    this.message = this.timestamp() + ' Loading game board\n'
    this.GetConfiguration()
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.board {
  display: grid;
  grid-template-columns: repeat(12, 41.5px);
  grid-auto-rows: 30px;
  width: 500px;
  display: grid;
  float: left;
}
button {
  color: white;
  border-radius:4px!important;
}
.slot {
  color: white;
}
.stats {
  display: grid;
  border-collapse: collapse;
  table-layout: fixed;
  width: 100%;
  responsive: true;
}
th, td {
  width: 20px;
  border: 1px solid black;
  text-align: center;
  border-radius:4px!important;
}
.cell-cold{
  background-color: LightSkyBlue;
}
.cell-hot{
  background-color: Salmon;
}
input {
  width: 400px;
}
.log{
  border-radius:4px!important;
  background-color: Gray;
  border: 2px solid #993333;
  width: 400px;
  float: left;
  margin-top: 50px;
  position:relative;
}
.gameboard{
  border-radius: 25px;
  border: 2px solid #993333;
  background-color: Lime;
  width: 700px;
  height: 225px;
  padding: 10px 10px;
  display: inline-block;
  float: left;
  margin-right: 100px;
}
.spinner{
   float: right;
   animation: rotation 2s infinite linear;
}
.mains{
  padding: 50px 20px;
  background-color: LightBlue;
}
.wheelSpin{
 width:150px;
 text-align: center;
 animation: rotate 4s linear infinite;
}
.wheelStill{
 width:150px;
 text-align: center;
}
.eventDiv{
  font-align: top, center;
  float: left;
  border: 5px;
  width: 300px;
  clear:both;
  margin-top: 50px;
  margin-right: 500px;
  background-color: Wheat;
}
.eventTr{
  display: grid;
  height: 35px;
  background-color: Azure;
  border: 2px solid #FFF8DC;
  text-top-align: center;
}
.eventTable{
  border: 2px solid #FFF8DC;
  display: grid;
  width: 100%;
  border-collapse: collapse;
  table-layout: fixed;
  background-color: Azure;
}
@keyframes rotate {
  to {
    transform: rotate(360deg);
  }
}
</style>
