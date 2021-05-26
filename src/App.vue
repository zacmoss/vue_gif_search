<template>
  <h1>Giphy Search</h1>
  <div id="input_container">
    <input @input="debounceSearch" v-model="search_string" placeholder="search gifs">
    <button @click="submit">Submit</button>
  </div>
  <div v-if="this.error_message" class="error_message_container">
    <div class="error_message">
      <p><strong>{{ this.error_message }}</strong></p>
    </div>
  </div>
  
  <div style="height: 20px">
  <p v-if="search_string && !this.error_message"><strong>Searching the interwebs for cool gifs matching: {{ search_string }}</strong></p>
  </div>
  <div class="row">
    <div class="loader" v-if="loading">
      <p>Loading...</p>
    </div>
    <div v-else-if="this.display_gif" :class="test">
      <div style="display:flex; justify-content: space-between; margin-bottom: 1rem">
        <button @click="this.index > 0 ? cycle('-') : null">Previous</button>
        <button @click="this.index + 1 < this.displayed_gifs.length ? cycle('+') : null">Next</button>
      </div>
      <div>
        <video width="320" height="240" autoplay muted loop :key="this.display_gif">
          <source :src="this.display_gif.images.original_mp4.mp4" type="video/mp4">
        </video>
      </div>
    </div>
  </div>
  <div class="footer">
    <img src="./assets/giphy_logo.png">
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'App',
  data: function () {
    return {
      giphy_api_key: undefined,
      displayed_gifs: [],
      display_gif: undefined,
      search_string: "",
      random: true,
      baseURL: "https://api.giphy.com/v1/gifs/",
      giphyURL: "",
      keys: [],
      stored_gifs: [],
      test: "one_col",
      loading: false,
      debounce: null,
      too_many_requests: false,
      error_message: '',
      index: 0
    }
  },
  mounted: function() {
    // pull in env vars (must have VUE_APP_ prefix to work with webpack)
    this.giphy_api_key = process.env.VUE_APP_GIPHY_API_KEY

    this.getGifs()
    
  },
  watch: {
    too_many_requests: function (val) {
      // if too_many_requests changes and is true, set error message
      if (val) {
        this.error_message = "Api request limit has been reached."
      }
    }
  },
  methods: {
    submit: function () {
      this.setRandom() // do we even have to set it here? it would have been set on keystroke stop
      this.getGifs()
    },
    formatSearch: function() {
      if (this.random) {
        this.giphyURL = encodeURI(this.baseURL + "random?api_key=" + this.giphy_api_key + "&limit=5")
      } else {

        let formatted_string = this.search_string.replace(/[^\w\s/d+1/]/gi, '')
        formatted_string = this.search_string.split(' ').join('+')

        this.giphyURL = encodeURI(this.baseURL + "search?q="+formatted_string+"&api_key="+this.giphy_api_key+"&limit=5")
      }      
    },
    debounceSearch() {
      clearTimeout(this.debounce)
      this.debounce = setTimeout(() => {
        this.setRandom()
        this.getGifs()
      }, 600)
    },
    getGifs: function () {
      this.loading = true
      this.formatSearch()
      if (this.random) {
        this.getThreeRandomGifs()
      } else {
        axios.get(this.giphyURL)
        .then((response) => {
          // TODO - Test this with test var
          if (response.data.meta.status === 200) {
            this.displayed_gifs = [] // maybe make this a helper function that can be called multiple places
            for (let i = 0; i < response.data.data.length; i++) {
              this.storeData(response.data.data[i])
            }
            this.index = 0
            this.setDisplayedGif()
          } else if (response.data.meta.status === 429) {
            this.too_many_requests = true
          } else {
            this.error_message = 'There was an error with that request.'
            console.log('ERROR: Request to giphy api returned code ' + response.data.meta.status)
          }
        
        this.loading = false
      });
      }
    },
    getThreeRandomGifs: function () {
      if (this.random) {
        this.displayed_gifs = []
        this.index = 0
        for (let i = 0; i < 3; i++) {
          this.getRandomGif()
        }
      } else {
        console.log('ERROR: Attempting to hit random gif url when not set to random')
      }
    },
    getRandomGif: function () {
      if (this.random) {
        axios.get(this.giphyURL)
        .then((response) => {
          this.storeData(response.data.data)
          this.setDisplayedGif()        
          this.loading = false
        });
      } else {
        console.log('ERROR: Attempting to hit random gif url when not set to random')
      }
    },
    requestGifs: function () {
      axios.get(this.giphyURL)
      .then((response) => {
        this.raw_data = response.data          
      });
    },
    formatData: function () {
    },
    storeData: function (data) {
      let key = data.images.original_mp4.mp4
              
      // if the gif url doesn't already exist in the locally stored gifs, add it to the array of gifs
      // the need for this.keys needs to be refactored, has to be a way to loop through this.stored_gifs keys to not be redundant
      if (!this.keys.includes(key)) {
        console.log('pushing data')
        this.displayed_gifs.push(data)
        this.stored_gifs[key] = data
        this.keys.push(key)
      } else { // the key already exists locally
        this.displayed_gifs.push(this.stored_gifs[key])
        this.keys.push(key)
      }
    },
    setRandom: function () {
      this.random = this.search_string ? false : true
    },
    setDisplayedGif: function () {
      console.log('setDisplayedGif')
      console.log(this.displayed_gifs[this.index])
      this.display_gif = this.displayed_gifs[this.index]
    },
    cycle: function (operator) {
      console.log('at cycle')
      console.log(this.index)
      if (operator === '+' && this.index + 1 < this.displayed_gifs.length) {
        this.index++
      } else if (operator === '-' && this.index > 0) {
        this.index--
      }
      console.log(this.index)
      this.setDisplayedGif()
    }
  }
}
</script>

<style>
input {
  margin: 0 .5rem;
  font-size: 16px;
  padding: 5px 10px;
  outline: none;
}
button {
  margin: 0 .5rem;
  color: white;
  background-color: #2c3e50;
  /*width: 50px;*/
  /*height: 25px;*/
  border: none;
  color: white;
  padding: 5px 10px;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 16px;
  margin: 4px 2px;
  cursor: pointer;
}
iframe {
  width: 200px;
}
video {
  /*border: 1px black solid;
  padding: 5px 5px;*/
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  min-height: 100vh;
  position: relative;
}
/*
#page_container {
  min-height: 100vh;
  position: relative;
}
*/
#input_container {
  margin-bottom: 2rem;
}
/* if default, width 30%, if mobile width 80% */
.error_message_container {
  display: flex;
  justify-content: center;
}
.error_message {
  color: #DD3444;
  border: 1px #DD3444 solid;
  /*background-color: rgba(240, 240, 240);*/
  border-radius: 3px;
  width: 25%;
  padding: 10px 0;
  font-size: 18px;
}
.error_message p {
  padding: 0; 
  margin: 0;
}
@media only screen and (max-width: 800px) {
  .error_message {
    width: 80%;
  }
}

.row {
  display: flex;
  justify-content: center;
  /*border: 1px solid black;*/
  margin: 4rem 0;
}
/*
@media only screen and (max-width: 1245px) {
  .row {
    display: block;
    margin-bottom: 4rem;
  }
}
*/
.five_col {
  flex: 10%;
  padding: 5px;
  /*border: 1px red solid;*/
}
.three_col {
  flex: 30%;
  padding: 5px;
  /*border: 1px blue solid;*/
}
.one_col {
  /*width: 400px;*/
}
.loader {
  flex: 100%;
}
.footer
{
  display: flex;
  justify-content: center;
  position: absolute;
  bottom:50px;
  width: 100%;
}
@media only screen and (max-width: 1245px) {
  .footer {
    position: static;
  }
}
</style>
