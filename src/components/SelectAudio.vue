<template>
  <div class="item">
    <h3>音频波形显示demo</h3>
    <section class="upload_box" id="upload3">
      <div class="upload_button_box">
        <!-- accept限制只能选择音频 -->
        <el-upload
          ref="upload"
          class="upload-demo"
          :limit="1"
          :on-exceed="handleExceed"
          :auto-upload="false"
          accept="audio/*"
          :on-change="showAudio"
          :on-remove="remove"
        >
          <template #trigger>
            <el-button type="primary">选择音频文件</el-button>
          </template>
          <template #tip>
            <div class="el-upload__tip text-red">
              限制1个音频，新文件将覆盖旧文件
            </div>
          </template>
        </el-upload>
      </div>
      <div class="upload_abbre">
        <audio
          :src="BlobUrl"
          v-show="isShow"
          controls
          @play="play"
          @pause="pause"
          @timeupdate="timeupdate"
        ></audio>
      </div>
    </section>
  </div>

  <div id="waveform" ref="waveform" v-show="isWaveform"></div>
  <!-- <button @click="playMusic" style="margin-top: 50px;">播放</button> -->
</template>

<script setup>
import { ref, watch, onMounted } from "vue";
import { ElButton, ElUpload } from "element-plus";
import WaveSurfer from "wavesurfer.js";
//波形框显示
let isWaveform=ref(false)


const upload = ref();
const handleExceed = (files) => {
  //超出一个文件就清空文件列表
  upload.value.clearFiles();
  let file = files[0];
  //   console.log("文件1", file);
  //手动选择文件
  upload.value.handleStart(file);
  // console.log("upload1",upload.value);
  wavesurfer.value.empty();
};
//移除文件将audio框消失
const remove = () => {
  isShow.value = false;
  wavesurfer.value.empty();
  isWaveform.value = false;
};

let waveform = ref();     //wavesurfer绘画框
let wavesurfer = ref();   //wavesurfer实例
//显示audio框
let isShow = ref(false);
let BlobUrl = ref("");
const showAudio = async (files) => {
  let file = files.raw;
  let BASE64 = await changeBASE64(file);
  //转blob对象
  let auidoBlob = dataURLtoBlob(BASE64);
  //创建blob url
  let blobUrl = window.URL.createObjectURL(auidoBlob);
  BlobUrl.value = blobUrl;
  isShow.value = true;
  isWaveform.value = true;
  // console.log("BlobUrl", BlobUrl.value);
};

onMounted(() => {
  wavesurfer.value = WaveSurfer.create({
    container: waveform.value,
    waveColor: "#00FF00",
    //波纹宽度
    // barWidth: 1,
    //光标的填充颜色，指示播放头的位置
    cursorColor: "#fff",
    //光标后面的波形部分的填充色.
    progressColor: "#7CFC00",
    backend: "MediaElement",
    // mediaControls: false,
    // 音频播放速度，数值越小越慢
    audioRate: "1",
  });
});

//音频的时间
let Audiotime;
let titleTime;
const timeupdate=async (e)=>{
  Audiotime=e.target.currentTime;  //获取音频当前的时间
  titleTime= e.target.duration;     //获取音频总时间的时间,单位为秒
  let progress=Audiotime/titleTime
  if(isNaN(progress)){
    progress=0
  }
  //波形根据音频时间跳转0-1之间
  wavesurfer.value.seekTo(progress)
}

watch(
  BlobUrl,
  () => {
    wavesurfer.value.load(BlobUrl.value);
  },
  {
    deep: true,
  }
);

//监听播放，暂停pause
const play = () => {
  wavesurfer.value.playPause.bind(wavesurfer.value)();
  wavesurfer.value.setVolume(0); //静音
};
const pause = () => {
  wavesurfer.value.playPause.bind(wavesurfer.value)();
};

// 把选择的文件读取成为BASE64
const changeBASE64 = (file) => {
  return new Promise((resolve) => {
    let fileReader = new FileReader();
    fileReader.readAsDataURL(file);
    fileReader.onload = (ev) => {
      resolve(ev.target.result);
    };
  });
};

//base64转blob对象，创建blob url
const dataURLtoBlob = (dataurl) => {
  var arr = dataurl.split(","),
    mime = arr[0].match(/:(.*?);/)[1],
    bstr = atob(arr[1]),
    n = bstr.length,
    u8arr = new Uint8Array(n);
  while (n--) {
    u8arr[n] = bstr.charCodeAt(n);
  }
  return new Blob([u8arr], {
    type: mime,
  });
};

</script>

<style scoped>
#waveform {
  height: 128px;
  background-color: rgba(0, 0, 0, 0.9);
}
</style>