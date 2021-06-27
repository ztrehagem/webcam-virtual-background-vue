<template>
  <div>
    <canvas ref="canvas" width="640" height="360"></canvas>
  </div>
</template>

<script lang="ts">
import { defineComponent, onMounted, ref } from "vue";
import { SelfieSegmentation } from "@mediapipe/selfie_segmentation";
import { Camera } from "@mediapipe/camera_utils";

export default defineComponent({
  setup(props, ctx) {
    const video = document.createElement('video')
    const canvas = ref<HTMLCanvasElement | null>(null)
    let canvasCtx: CanvasRenderingContext2D | null = null

    onMounted(() => {
      canvasCtx = canvas.value?.getContext('2d') ?? null

      const camera = new Camera(video, {
        onFrame: async () => {
          await selfieSegmentation.send({ image: video })
        },
        width: 640,
        height: 360
      });
      camera.start();
    })

    const selfieSegmentation = new SelfieSegmentation({
      locateFile: (path) => `https://cdn.jsdelivr.net/npm/@mediapipe/selfie_segmentation/${path}`
    })

    selfieSegmentation.setOptions({
      modelSelection: 1
    })

    selfieSegmentation.onResults((results) => {
      if (!canvas.value || !canvasCtx) return

      canvasCtx.save()
      canvasCtx.clearRect(0, 0, canvas.value.width, canvas.value.height);

      canvasCtx.drawImage(results.segmentationMask, 0, 0, canvas.value.width, canvas.value.height);

      // Only overwrite existing pixels.
      // canvasCtx.globalCompositeOperation = 'source-in';
      canvasCtx.globalCompositeOperation = 'source-out';
      canvasCtx.fillStyle = '#00FF00';
      canvasCtx.fillRect(0, 0, canvas.value.width, canvas.value.height);

      // Only overwrite missing pixels.
      canvasCtx.globalCompositeOperation = 'destination-atop';
      canvasCtx.drawImage(results.image, 0, 0, canvas.value.width, canvas.value.height);

      canvasCtx.restore();
    })

    return {
      canvas,
    }
  }
})
</script>
