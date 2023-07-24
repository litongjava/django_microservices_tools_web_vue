<template>
<div>
  <label>
    URL:
    <input v-model="url" type="text" size="255"/>
  </label>
  <br/>
  <label>
    Format:
    <input v-model="format" type="text"/>
  </label>
  <br/>
  <button @click="submit">Submit</button>
  <div>
    <h5>{{this.filename}}</h5>
  </div>

  <div v-if="audioUrl">
    <audio ref="audioPlayer" :src="audioUrl" controls></audio>
  </div>
  <div v-if="audioUrl">
    <button @click="playAudio">Play</button>
    <button @click="pauseAudio">Pause</button>
  </div>
  <div v-if="audioUrl">
    <button @click="download">download</button>
  </div>
</div>
</template>

<script>
import axios from 'axios';
import FormData from 'form-data';

export default {
  data() {
    return {
      serverUrl: 'https://tools-api-49f0fcf5538b.herokuapp.com',
      url: 'https://www.youtube.com/watch?v=HoE3bN5q7HA',
      format: '139',
      audioUrl: undefined,
      filename: undefined,
      uri: undefined
    };
  },
  methods: {
    playAudio() {
      this.$refs.audioPlayer.play();
    },
    pauseAudio() {
      this.$refs.audioPlayer.pause();
    },
    decodeMimeFilename(mimeEncodedFilename) {
      const startOfFilename = mimeEncodedFilename.indexOf('?b?');
      const endOfFilename = mimeEncodedFilename.lastIndexOf('?=');

      if (startOfFilename === -1 || endOfFilename === -1) {
        return null;
      }

      const base64EncodedFilename = mimeEncodedFilename.slice(startOfFilename + 3, endOfFilename);
      const filenameBytes = atob(base64EncodedFilename);

      let filename = '';
      for (let i = 0; i < filenameBytes.length; i++) {
        filename += '%' + ('0' + filenameBytes.charCodeAt(i).toString(16)).slice(-2);
      }

      return decodeURIComponent(filename);
    },
    submit() {
      const data = {
        url: this.url,
        format: this.format
      };


      const config = {
        method: 'get',
        //url: 'https://tools-api-49f0fcf5538b.herokuapp.com/youtube/download',
        // url: 'http://127.0.0.1:8000/youtube/download',
        url: this.serverUrl + "/youtube/download",
        params: data,
      };

      axios(config)
        .then(response => {
          this.filename = response.data.data.filename.split('/').pop();
          this.audioUrl = this.serverUrl + "/youtube/download_file?filename=" + response.data.data.filename
        })
        .catch(error => {
          console.log(error);
        });
    },
    download() {
      const config = {
        method: 'get',
        url: this.audioUrl,
        responseType: 'blob' // make sure to quote blob
      };
      axios(config)
        .then(response => {
          // Create a blob from the response
          const url = window.URL.createObjectURL(new Blob([response.data]));
          const link = document.createElement('a');
          link.href = url;
          const contentDisposition = response.headers['content-disposition'];
          let fileName = this.filename;
          if (contentDisposition) {
            const fileNameMatch = contentDisposition.match(/filename="(.+)"/);
            if (fileNameMatch.length === 2)
              fileName = fileNameMatch[1];
          }
          link.setAttribute('download', fileName);
          document.body.appendChild(link);
          link.click();
          document.body.removeChild(link);
        })
        .catch(error => {
          console.log(error);
        });
    }

  },
};
</script>
