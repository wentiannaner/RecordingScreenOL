<template>
  <div id="app">
    <div class="container">
      <div class="row justify-content-center">
        <div class="col-xl-auto">
          <div class="card">
            <div class="card-header">屏幕录制</div>
            <div class="card-body">
              <div class="row">
                <div class="col-xl">
                  <div class="form-check form-check-inline">
                    <input type="checkbox" v-model="checked" class="form-check-input" id="audio" :disabled="isStart && !isFinish" />
                    <label for="audio" class="form-check-label">是否录制声音</label>
                  </div>
                </div>
              </div>
              <div class="row">
                <div class="col-xl">
                  <div class="card">
                    <video ref="video" src width="600" height="400"></video>
                  </div>
                </div>
              </div>
              <div class="row">
                <div class="col-xl-auto">
                  <button class="btn btn-primary" :disabled="isStart" @click="onStartBtn">开始录制</button>
                </div>
                <div class="col-xl-auto">
                  <button class="btn btn-danger" :disabled="!isStart" @click="onEndBtn">结束录制</button>
                </div>
                <div class="col-xl-auto">
                  <button class="btn btn-info" :disabled="!isFinish" @click="onDownloadBtn">下载</button>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>

export default {
  name: 'app',
  data () {
    return {
      mediaRecorder: null,
      data: [],
      stream: null,
      checked: false,
      isStart: false,
      isFinish: false
    }
  },
  methods: {
    async onStartBtn () {
      this.isStart = true
      this.isFinish = false
      this.stream = await navigator.mediaDevices.getDisplayMedia({
        video: true,
        audio: this.checked
      })
      this.mediaRecorder = new MediaRecorder(this.stream, {
        mimeType: 'video/webm'
      })
      this.mediaRecorder.ondataavailable = e => {
        this.data.push(e.data)
      }
      this.mediaRecorder.start()
      const video = this.$refs['video']
      video.srcObject = this.stream
      video.play()
    },
    onEndBtn () {
      this.isStart = false
      this.isFinish = true
      this.$refs['video'].pause()
      this.mediaRecorder.pause()
      this.mediaRecorder.stop()
      this.stream.getTracks().forEach(track => {
        track.stop()
      })
    },
    onDownloadBtn () {
      let res = new Blob(this.data, { type: 'video/webm' })
      let url = URL.createObjectURL(res)
      let a = document.createElement('a')
      a.href = url
      a.download = `${new Date().getTime()}.webm`
      a.click()
      a.remove()
    }
  }
}
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 60px;
}

.row {
  margin-bottom: 10px;
}
</style>
