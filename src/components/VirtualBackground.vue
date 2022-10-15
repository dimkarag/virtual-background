<template>
  <div id="container">
    <img v-show="showBackground" id="background-image" :src="backgroundImageSrc"/>
    <video id="video" playsinline autoplay hidden muted @ended="onVideoEnd"></video>
    <img id="image" hidden />
    <canvas id="out-canvas"></canvas>
  </div>
</template>

<script>
import '@tensorflow/tfjs-backend-webgl';
const bodyPix = require('@tensorflow-models/body-pix');
export default {
  name: 'App',
  props: {
    backgroundImageSrc: {
      type: String,
      required: false
    },
    contentSrc: {
      type: String,
      required: true
    },
    fps: {
      type: Number,
      required: false,
      default: () => {
        return 30
      }
    },
    removalType: {
      type: String,
      default: () => {
        return 'personSegmentation'
      }
    },
    chromaColorConfig: {
			type: Object,
				default: () => {
					return {
						Rgt: 10,
						Ggt: 80,
						Blt: 30
				}
			}
    },
    bodyPixConfig: {
      type: Object,
      required: false,
      default: () => {
        return {
          internalResolution: 0.2,
          segmentationThreshold: 0.5,
          scoreThreshold: 0.5
        }
      }
    },
    personSegmentationConfig: {
      type: Object,
      required: false,
      default: () => {
        return {
          architecture: 'MobileNetV1',
          outputStride: 16,
          multiplier: 1,
          quantBytes: 4
        }
      }
    }
  },
  data() {
    return {
      typeOfSrc: null,
      videoTags: ['mp4', 'wav'],
      imageTags: ['jpeg', 'jpg', 'png'],
      canvas: null,
      srcElement: null,
      net: null,
      context: null,
      contextTemp: null,
      width: 0,
      height: 0,
      fpsInterval: 0,
      showBackground: false,
      fpsTimers: {
          now: 0,
          then: 0,
          elapsed: 0
      },
			chromaColorOptions: {},
      reqId: 0
    }
  },
  created() {
		if (this.removalType === 'chromaColor') {
			this.R_gt = this.chromaColorConfig['Rgt'],
			this.G_gt = this.chromaColorConfig['Ggt'],
			this.B_lt = this.chromaColorConfig['Blt'],
			this.R_lt = this.chromaColorConfig['Rlt'],
			this.G_lt = this.chromaColorConfig['Glt'],
			this.B_gt = this.chromaColorConfig['Bgt']
		}
  },
  async mounted() {
		this.net = await bodyPix.load(this.bodyPixConfig)
		this.init()
  },
  methods: {
    init() {
			const srcArray = this.contentSrc.split(".")
			const fileType = srcArray[srcArray.length - 1]
			if (this.videoTags.includes(fileType)) {
				this.renderVideo()
			}
			else if (this.imageTags.includes(fileType)) {
				this.renderImage()
			}
    },
    renderImage() {
      this.typeOfSrc = 'image'
      this.srcElement = document.getElementById('image');
      const image = new Image()
      image.src = this.contentSrc
      this.srcElement.src = image.src
      image.onload = () => {
        this.width = image.width
        this.height = image.height
        this.initCanvas()
        this.draw()
      }
    },
    renderVideo() {
      this.fpsInterval = 1000 / this.fps
      this.typeOfSrc = 'video'
      this.srcElement = document.getElementById('video');
      this.srcElement.src = this.contentSrc
      this.srcElement.onloadeddata = () => {
        this.width = this.srcElement.videoWidth
        this.height = this.srcElement.videoHeight
        this.initCanvas()
        this.fpsTimers.then = Date.now()
        this.animate()
        this.srcElement.play()
      }
    },
    initCanvas() {
      this.canvas = document.getElementById('out-canvas');
      this.canvas.setAttribute("width", this.width)
      this.canvas.setAttribute("height", this.height)
      this.context = this.canvas.getContext("2d")
      this.canvasTemp = document.createElement("canvas")
      this.canvasTemp.setAttribute("width", this.width)
      this.canvasTemp.setAttribute("height", this.height)
      this.contextTemp = this.canvasTemp.getContext("2d")
      this.container = document.getElementById('container');
      if (!this.backgroundImageSrc) {
        this.canvas.style.position = 'relative'
      }
    },
    animate() {
      this.reqId = requestAnimationFrame(this.animate)
      const now = Date.now()
      this.fpsTimers.elapsed = now - this.fpsTimers.then
      if (this.fpsTimers.elapsed > this.fpsInterval) {
          this.fpsTimers.then = now - (this.fpsTimers.elapsed % this.fpsInterval)
          this.draw()
      }
    },
    async draw() {
      this.contextTemp.clearRect(0, 0, this.width, this.height)
      this.contextTemp.drawImage(this.srcElement, 0, 0, this.width, this.height)
      if (this.removalType === 'personSegmentation') {
        this.backgroundRemoval()
      } else if (this.removalType === 'chromaColor') {
        this.chromaColorRemoval()
      }
    },
    onVideoEnd() {
      cancelAnimationFrame(this.reqId)
    },
    async backgroundRemoval() {
      const imgData = this.contextTemp.getImageData(0, 0, this.width, this.height)
      const segmentation = await this.net.segmentPerson(this.canvasTemp, this.personSegmentationConfig)
      this.context.clearRect(0,0, this.width, this.height)
      for(let i=0; i < segmentation.data.length; i++) {
        if (segmentation.data[i] === 0) {
          imgData.data[i * 4 + 3] = 0
        }
      }
      this.context.putImageData(imgData, 0, 0)
      if (!this.showBackground) {
        this.showBackground = !this.showBackground
      }
    },
		shouldRemoveColor(r, g, b) {
			if((this.R_gt < r || this.R_lt > r) &&
				(this.G_gt < g || this.G_lt > g) &&
				(this.B_gt < b || this.B_lt > b)) {
				return true
			}
			return false
		},
    async chromaColorRemoval() {
      const imgData = this.contextTemp.getImageData(0, 0, this.width, this.height)
      this.context.clearRect(0,0, this.width, this.height)
      for (let i=0; i < imgData.data.length; i+=4) {
        let r = imgData.data[i]
        let g = imgData.data[i + 1]
        let b = imgData.data[i + 2]
        if (this.shouldRemoveColor(r, g, b)) {
          imgData.data[i + 3] = 0
        }
      }
      this.context.putImageData(imgData, 0, 0)
      if (!this.showBackground) {
        this.showBackground = !this.showBackground
      }
    }
  }
}
</script>
<style scoped>
#background-image {
  width: 100%;
  position: relative;
}

#out-canvas {
  width: 100%;
  position: absolute;
  left: 0;
  bottom: 0;
}

#container {
  width: 100%;
  position: relative;
}
</style>
