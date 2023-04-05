<template>
  <div class="hello">
    <p>{{text}}</p>
    <div class='input'>
      <label for='input'>( input )</label>
      <select v-model='inputLanguage' id='input'>
        <option v-for='(language, index) in languages' :key='index' :value='language.code'>
          {{language.name}}
        </option>
      </select>
    </div>
    <div class='output'>
      <select v-model='outputLanguage' id='output'>
        <option v-for='(language, index) in languages' :key='index' :value='language.code'>
          {{language.name}}
        </option>
      </select>
      <label for='output'>( output )</label>
    </div>
    <button @touchstart='recordStart' @touchend='recordStop' class='recordButton'>録音</button>
  </div>
</template>

<script>
import axios from 'axios'
import languages from '../assets/languages.json'

export default {
  name: 'HelloWorld',
  data() {
    return {
      text: '',
      mediaRecorder: null,
      chunks: [],
      languages: languages,
      inputLanguage: 'ja',
      outputLanguage: 'en'
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
      const url = 'https://api.openai.com/v1/audio/transcriptions'
      const data = new FormData()
      data.append('model', 'whisper-1')
      data.append('file', blob, 'recording.mp3')
      data.append('language', this.inputLanguage)

      await axios.post(url, data, { headers: {
        'Authorization': `Bearer ${process.env.VUE_APP_WHISPER_API_KEY}`,
        'Content-Type': 'multipart/form-data'
      } })
          .then(response => this.textTranslation(response.data.text))
          .catch(error => console.log(error))
    },
    async textTranslation(text) {
      const url = 'https://script.google.com/macros/s/AKfycby05LCeZMPERoqQc-aAUzxwyDoCv24GNluE69MBrp4IuZjBLSF1q6cN68zo4GL-3hQ/exec'

      await axios.get(url, {
        params: {
          text: text,
          source: this.inputLanguage,
          target: this.outputLanguage
        }
      })
        .then(response => this.text = response.data.text)
        .catch(error => console.log(error))
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.input {
  position: fixed;
  bottom: 0;
  left: 0;
  margin: 1rem;
}

.output {
  position: fixed;
  bottom: 0;
  right: 0;
  margin: 1rem;
}

.recordButton {
  position: fixed;
  top: 0;
  left: 0;
  margin: 0.5rem;
  height: 80vh;
}
</style>
