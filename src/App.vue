<template>
  <div class="container">
    <button @click="requestFullscreen">全屏</button>
    <div class="video-container" ref="videoContainerRef" @click="startCam" @dblclick="closeVideo">
      <video ref="video" :style="`transform: scaleX(${hFlip ? -1 : 1})`"></video>
      <div class="overlay" v-show="!isPlaying">
        <span>点击打开摄像头</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue';

const hFlip = ref(true)
const videoContainerRef = ref<HTMLDivElement>();
const videoRef = ref<HTMLVideoElement | null | undefined>();


const isFullscreen = ref(false);
const isPlaying = ref(false);
const mediaStream = ref<MediaStream | null>();
const isInitialized = ref(false);

const requestFullscreen = async () => {
  const videoContainer = videoContainerRef.value;
  isFullscreen.value = true;
  if (videoContainer?.requestFullscreen) {
    await videoContainer.requestFullscreen();
  }
};

const startCam = async () => {
  if (isPlaying.value || isInitialized.value) {
    return;
  }
  try {
    mediaStream.value = await navigator.mediaDevices.getUserMedia({ video: true });
    videoRef.value!.srcObject = mediaStream.value;
    isInitialized.value = true;
    playVideo();
  } catch (error) {
    console.log(error);
  }
};

const playVideo = () => {
  isPlaying.value = true;
  videoRef.value!.play();
};

const pauseVideo = () => {
  isPlaying.value = false;
  videoRef.value!.pause();
};

const closeVideo = () => {
  pauseVideo();
  mediaStream.value!.getTracks()[0].stop();
  mediaStream.value = null;
  isInitialized.value = false;
};

onMounted(() => {
  videoRef.value = document.querySelector('.video-container video') as HTMLVideoElement | null | undefined;
});

onUnmounted(() => {
  mediaStream.value!.getTracks()[0].stop();
  mediaStream.value = null;
});
</script>

<style>
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 6px;
}

.video-container {
  width: 320px;
  height: 320px;
  background-color: #000;
  position: relative;
  cursor: pointer;
  /* 将鼠标指针设置为手型，以表示该元素可点击 */
}

video {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.7);
  display: flex;
  justify-content: center;
  align-items: center;
  color: #fff;
  font-size: 24px;
  cursor: pointer;
}
</style>