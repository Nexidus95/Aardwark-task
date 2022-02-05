<template>
 <div>
  <div class="hello">
  </div>
  <div>
    <span>Multiline message is:</span>
    <p style="white-space: pre-line;">{{ message }}</p>
    <br>
  </div>
  <div>
    <li v-for="won in mostwon" v-bind:key="won">{{won}}</li>
  </div>
 </div>
</template>

<script>
export default {
  data () {
    return {
      message: '',
      mostwon: {}
    }
  },
  methods: {
    MostWon () {
      this.message = this.message + this.timestamp() + ' GET ../configuration\n'
      fetch('https://dev-games-backend.advbet.com/v1/ab-roulette/1/configuration')
        .then(response => response.json())
        .then((data) => {
          this.mostwon = data.positionToId
        })
    },
    timestamp () {
      const current = new Date()
      const date = current.getFullYear() + '-' + (current.getMonth() + 1) + '-' + current.getDate() + 'T'
      const time = current.getHours() + ':' + current.getMinutes() + ':' + current.getSeconds() + '.' + current.getMilliseconds() + 'Z'
      const dateTime = date + time
      return dateTime
    }
  },
  mounted () {
    this.message = this.timestamp() + ' App has started\n'
    this.MostWon()
  }
}
</script>
