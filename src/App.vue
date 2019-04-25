<template>
  <div id="app">
    <router-view />
    <a-modal title="更新进度" v-model="showUpdater">
      <div>
        {{ downloadProcess.transferred }}
      </div>
      <div>
        {{ downloadProcess.total }}
      </div>
      <div>
        {{ downloadProcess.bytesPerSecond }}
      </div>
      <div>
        {{ downloadProcess.percent }}
      </div>
    </a-modal>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import Component from 'vue-class-component'
import { ipcRenderer } from 'electron'

@Component
export default class App extends Vue {
  downloadProcess = {}

  showUpdater = false

  created() {
    ipcRenderer.once('checking-for-update', (event: Event, data: any) => {
      console.log('data', data)
    })
    ipcRenderer.once('update-available', (event: Event, data: any) => {
      console.log('update-available', data)
      this.$confirm({
        title: '发现新版本，是否更新？',
        content: `${data.version}`,
        onOk() {
          console.log('点击了 OK')
          ipcRenderer.send('autoUpdater-toDownload')
        },
        onCancel() {},
      })
    })

    // 下载进度
    ipcRenderer.on('download-progress', (event: Event, process: any) => {
      if (process.transferred >= 1024 * 1024) {
        process.transferred = `${(process.transferred / 1024 / 1024).toFixed(2)}M`
      } else {
        process.transferred = `${(process.transferred / 1024).toFixed(2)}K`
      }
      if (process.total >= 1024 * 1024) {
        process.total = `${(process.total / 1024 / 1024).toFixed(2)}M`
      } else {
        process.total = `${(process.total / 1024).toFixed(2)}K`
      }
      if (process.bytesPerSecond >= 1024 * 1024) {
        process.speed = `${(process.bytesPerSecond / 1024 / 1024).toFixed(2)}M/s`
      } else if (process.bytesPerSecond >= 1024) {
        process.speed = `${(process.bytesPerSecond / 1024).toFixed(2)}K/s`
      } else {
        process.speed = `${process.bytesPerSecond}B/s`
      }
      process.percent = process.percent.toFixed(2)
      this.downloadProcess = process
      this.showUpdater = true
    })
  }
}
</script>

<style lang="less" scoped>

</style>


<style>

  /* Global CSS */
  body {
    background: #fff;
    color: #434343;
  }

  ::-webkit-scrollbar{
    width: 0px;
    height: 6px;
    border-radius: 4px;
    background-color: #fff;

  }
    /*滚动条两端的箭头*/
  ::-webkit-scrollbar-button{
    display: none;
  }
  ::-webkit-scroll-track{
    display: none;
  }
  ::-webkit-scrollbar-track-piece {
    display: none;
  }

  ::-webkit-scrollbar-thumb{
    background-color: #eee;
    opacity: 0.7;
    border-radius: 4px;
  }

  ::-webkit-scrollbar-corner {
    display: none;
  }
  ::-webkit-resizer{
    display: none;
  }

  .github {
    font-size: 16px;
    margin-left: 16px;
    cursor: pointer;
  }

  .logo {
    user-select: none;
  }

  .application {
    font-family: system-ui,-apple-system,BlinkMacSystemFont,Segoe UI,Roboto,Oxygen,Ubuntu,Cantarell,Droid Sans,Helvetica Neue,Fira Sans,sans-serif!important;
  }
</style>
