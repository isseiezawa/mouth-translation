<template>
  <div class="hello">
    <p>{{text}}</p>
    <button @touchstart='recordStart' @touchend='recordStop'>録音</button>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'HelloWorld',
  data() {
    return {
      text: '',
      mediaRecorder: null,
      chunks: [],
    }
  },
  mounted() {
    window.oncontextmenu = (e) => {
      e.preventDefault()
      e.stopPropagation()
      return false
    }
    this.recordSetup()
  },
  methods: {
    recordSetup() {
      if (navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
        console.log("getUserMedia supported.");
        navigator.mediaDevices
          .getUserMedia(
            // 制約 - 音声のみがこのアプリでは必要
            {
              audio: true,
            }
          )
          // 成功コールバック
          .then((stream) => {
            this.mediaRecorder = new MediaRecorder(stream)

            this.mediaRecorder.ondataavailable = (e) => {
              this.chunks.push(e.data)
            }

            this.mediaRecorder.onstop = () => {
              console.log(this.chunks)
              const blob = new Blob(this.chunks, {
                type: 'audio/mp3'
              })
              this.speechToText(blob)
              this.chunks = []
            }
          })
          // エラーコールバック
          .catch((err) => {
            console.error(`The following getUserMedia error occurred: ${err}`);
          });
      } else {
        console.log("getUserMedia not supported on your browser!");
      }
    },
    recordStart() {
      this.mediaRecorder.start()
    },
    recordStop() {
      this.mediaRecorder.stop()
    },
    async speechToText(blob) {
      console.log(blob)
      const url = 'https://api.openai.com/v1/audio/transcriptions'
      const data = new FormData()
      data.append('model', 'whisper-1')
      data.append('file', blob, 'recording.mp3')
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
