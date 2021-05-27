<template>
  <h1>Giphy Search</h1>
  <div id="input_container">
    <input @input="debounceSearch" v-model="search_string" placeholder="search gifs">
    <button @click="submit">Submit</button>
  </div>
  <div class="message_parent_container">
    <div class="message_child_container">
      <div v-if="this.error_message" class="error_message">
        <p><strong>{{ this.error_message }}</strong></p>
      </div>
      <div v-else-if="this.show_copied_message" class="copied_message">
        <p><strong>Copied to clipboard!</strong></p>
      </div>
      <div v-else-if="search_string">
        <p><strong>Searching the interwebs for cool gifs matching: {{ search_string }}</strong></p>
      </div>
    </div>
  </div>
  <div class="gif_parent_container">
  <div class="gif_container">
    <div class="loader" v-if="loading">
      <p>Loading...</p>
    </div>
    <div v-else-if="this.display_gif">
      <div class="cycle_container">
        <button @click="this.index > 0 ? cycle('-') : null">Previous</button>
        <button @click="this.index + 1 < this.displayed_gifs.length ? cycle('+') : null">Next</button>
      </div>
      <div>
        <video width="320" height="240" autoplay muted loop :key="this.display_gif">
          <source :src="this.display_gif.images.original_mp4.mp4" type="video/mp4">
        </video>
      </div>
      <div class="copy_to_clipboard_container">
      <button @click="this.copyToClipboard">Copy To Clipboard</button>
      </div>
    </div>
  </div>
  </div>
  <div class="footer_container">
  <div class="footer">
    <img src="./assets/giphy_logo.png">
  </div>
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
      loading: false,
      debounce: null,
      too_many_requests: false,
      error_message: '',
      index: 0,
      show_copied_message: false
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
    },
    display_gif: function () {
      this.show_copied_message = false
    }
  },
  methods: {
    submit: function () {
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
      this.show_copied_message = false
      this.loading = true
      this.formatSearch()
      if (this.random) {
        this.getThreeRandomGifs()
      } else {
        axios.get(this.giphyURL)
        .then((response) => {
          if (response.data.meta.status === 200) {
            this.displayed_gifs = []
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
    storeData: function (data) {
      let key = data.images.original_mp4.mp4
              
      // if the gif url doesn't already exist in the locally stored gifs, add it to the array of gifs
      // the need for this.keys needs to be refactored, has to be a way to loop through this.stored_gifs keys to not be redundant
      if (!this.keys.includes(key)) {
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
      this.display_gif = this.displayed_gifs[this.index]
    },
    cycle: function (operator) {
      if (operator === '+' && this.index + 1 < this.displayed_gifs.length) {
        this.index++
      } else if (operator === '-' && this.index > 0) {
        this.index--
      }
      this.setDisplayedGif()
    },
    copyToClipboard: function () {
      if (navigator.clipboard.writeText(this.display_gif.images.original_mp4.mp4)) {
        this.show_copied_message = true
      }
    }
  }
}
</script>

<style>
body {
  background-color: #333745;
}
input {
  margin: 0 .5rem;
  font-size: 16px;
  padding: 5px 10px;
  outline: none;
}
button {
  margin: 0 .5rem;
  color: white;
  background-color: #8457C5;
  border-bottom: 2px solid #6A3BAD;
  padding: 5px 10px;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 16px;
  cursor: pointer;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: white;
  margin-top: 3rem;
  min-height: 100vh;
  position: relative;
}
#input_container {
  margin-bottom: 2rem;
}
.message_parent_container {
  display: flex;
  justify-content: center;
  height: 2.5rem;
  margin: 2rem 0;
}
.message_child_container {
  width: 50%;
  font-size: 18px;
  text-align: center;
}
@media only screen and (max-width: 800px) {
  .message_child_container {
    width: 80%;
  }
}
.error_message {
  color: #DD3444;
}
.copied_message {
  color: white;
}
.gif_parent_container {
  display: flex;
  justify-content: center;
}
.gif_container {
  display: flex;
  justify-content: center;
  margin: 4rem 0;
  width: 500px;
  background-color: #464C5D;
  padding: 1rem 0;
  border: 3px gray solid;
  border-radius: 5px;
}
.cycle_container {
  display:flex; 
  justify-content: space-between; 
  margin-bottom: 1rem;
}
.copy_to_clipboard_container {
  margin-top: 1rem;
}
.loader {
  flex: 100%;
}
.footer
{
  display: flex;
  justify-content: center;
  position: absolute;
  bottom: 3rem;
  width: 100%;
}
@media only screen and (max-width: 1245px) {
  .footer {
    position: static;
  }
}
</style>
