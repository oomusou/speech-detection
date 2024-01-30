<template>
  <div>Speech Detection Demo</div>
  <div>
    <button @click="onStart">Start</button>
    <button @click="onStop">Stop</button>
  </div>
  <div>Threshold：<input type="text" v-model.number="threshold" /></div>
  <div>avgVolume: {{ avgVolume }}</div>
  <div>isSpeaking: {{ isSpeaking }}</div>
</template>

<script setup>
import { ref } from 'vue'
import { MediaRecorder } from 'extendable-media-recorder'

let threshold = ref(50)
let avgVolume = ref(0)
let isSpeaking = ref(false)

let mediaRecorder = null
let chunks = []

let onStart = async () => {
  const sampleRate = 16000
  const sampleInterval = 250 // 0.5s

  try {
    let mediaStream = await navigator.mediaDevices.getUserMedia({
      audio: true
    })

    let audioContext = new AudioContext({ sampleRate })
    let analyser = audioContext.createAnalyser()
    analyser.fftSize = 256

    let srcNode = new MediaStreamAudioSourceNode(audioContext, { mediaStream })
    let destNode = new MediaStreamAudioDestinationNode(audioContext)

    srcNode.connect(analyser)
    analyser.connect(destNode)

    mediaRecorder = new MediaRecorder(destNode.stream, {
      mimeType: 'audio/wav'
    })

    mediaRecorder.ondataavailable = (e) => {
      chunks.push(e.data)

      const bufferLength = analyser.frequencyBinCount
      let dataArray = new Uint8Array(bufferLength)
      analyser.getByteFrequencyData(dataArray)

      avgVolume.value = dataArray.reduce((sum, value) => sum + value, 0) / bufferLength
      isSpeaking.value = avgVolume.value > threshold.value
    }

    mediaRecorder.start(sampleInterval)
  } catch (err) {
    console.warn(err)
  }
}

let onStop = () => {
  mediaRecorder.stop()
}
</script>
