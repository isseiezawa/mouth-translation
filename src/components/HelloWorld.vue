<template>
  <div class="hello">
    <p>{{text}}</p>
    <button v-on:click='fetchApi'>api</button>
    <input type="file" id="file-input">
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'HelloWorld',
  data() {
    return {
      text: ''
    }
  },
  mounted() {
  },
  methods: {
    async fetchApi() {
      const url = 'https://api.openai.com/v1/audio/transcriptions'

      const fileInput = document.getElementById('file-input')
      const data = new FormData()
      data.append('model', 'whisper-1')
      data.append('file', fileInput.files[0])
      data.append('language', 'ja')

      await axios.post(url, data, { headers: {
        'Authorization': `Bearer ${process.env.VUE_APP_WHISPER_API_KEY}`,
        'Content-Type': 'multipart/form-data'
      } })
          .then(response => this.text = response.data.text)
          .catch(error => console.log(error))
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
