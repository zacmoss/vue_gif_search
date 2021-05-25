<template>
  <img alt="Vue logo" src="./assets/logo.png">
  <HelloWorld msg="Welcome to Your Vue.js App"/>
  <div>
    <input v-model="search_string" placeholder="search gifs">
    <button @click="submit"></button>
  </div>
  <p>Searching for: {{ search_string }}</p>
  <div>
    <div v-for="gif in displayed_gifs" :key="gif.embed_url">
      <iframe :src="gif.embed_url" frameBorder="0"></iframe>
      <!-- SHOULD THIS BE THE URL OR EMBED URL, NEED TO TEST WITH CHAT PROGRAM -->
      <p>{{ gif.url }}</p>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
import HelloWorld from './components/HelloWorld.vue'

export default {
  name: 'App',
  components: {
    HelloWorld
  },
  data: function () {
    return {
      giphy_api_key: undefined,
      displayed_gifs: [],
      search_string: "ryan+gosling",
      keys: [],
      stored_gifs: []
    }
  },
  mounted: function() {

    // pull in env vars (must have VUE_APP_ prefix to work with webpack)
    this.giphy_api_key = process.env.VUE_APP_GIPHY_API_KEY


    this.getGifs()

    // initial api call to get 3 random gifs to display
    console.log('mounted')
    
  },
  methods: {
    submit: function () {
      console.log('submitted')
      this.formatSearch()
      this.getGifs()
    },
    formatSearch: function() {
      /*
      let string = this.search_string
      let formatted_string = string.split(' ')
      let join = formatted_string.join('+')
      this.search_string = join
      */
      this.search_string = this.search_string.split(' ').join('+')
      console.log(this.search_string)
    },
    getGifs: function () {
      axios.get("http://api.giphy.com/v1/gifs/search?q="+this.search_string+"&api_key="+this.giphy_api_key+"&limit=5")
      .then((response) => {
        this.displayed_gifs = [] // maybe make this a helper function that can be called multiple places
        console.log(response.data);
        console.log(response.data.data.length)
        for (let i = 0; i < response.data.data.length; i++) {
          console.log(response.data.data[i])
          if (!this.keys.includes(response.data.data[i].embed_url)) {
            console.log('embed url not existing in keys')
            this.displayed_gifs.push(response.data.data[i])
            this.stored_gifs.push(response.data.data[i]) // THIS IS REDUDANT, FIX LOGIC
            this.keys.push(response.data.data[i].embed_url)
          } else {
            console.log('embed url exsits in keys already')
          }
        }
        console.log(this.gifs)
      });
    }
  }
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
</style>
