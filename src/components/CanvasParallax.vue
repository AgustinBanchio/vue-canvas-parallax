<template>
  <canvas id="parallax-canvas" class="parallax-canvas" :width="width" :height="height"></canvas>
</template>

<script>
export default {
  name: "CanvasParallax",
  props: {
    layers: {
      type: Array,
      default: () => [],
    },
    width: undefined,
    height: undefined,
    limitAngleX: {
      type: Number,
      default: 25,
    },
    limitAngleY: {
      type: Number,
      default: 25,
    },
    paddingX: {
      type: Number,
      default: 0,
    },
    paddingY: {
      type: Number,
      default: 0,
    },
  },
  data() {
    return {
      loadedLayers: [],
      canvas: undefined,
      context: undefined,
      ua: undefined,
      pointer: { x: 0, y: 0},
      pointerInitial: { x: 0, y: 0},
      motion: {x:0, y:0},
      motionInitial: {x: null, y: null},
      isMoving: false,
      totalX: 0,
      totalY: 0,
      maxOffset: 2000,
    }
  },
  computed: {
    isAndroid() {
     return this.ua.indexOf('android') > -1
    },
    maxMovementX() {
      return this.limitAngleX / 0.15;
    },
    maxMovementY() {
      return this.limitAngleY / 0.15;
    },
  },
  mounted() {
    this.loadLayers();
    this.canvas = document.getElementById('parallax-canvas')
    this.context = this.canvas.getContext('2d')
    this.ua = navigator.userAgent.toLowerCase()


    this.canvas.addEventListener('touchstart', this.pointerStart)
    this.canvas.addEventListener('mousedown', this.pointerStart)
    window.addEventListener('touchmove', this.pointerMove)
    window.addEventListener('mousemove', this.pointerMove)
    this.canvas.addEventListener('touchmove', (event)=>  { event.preventDefault() })
    this.canvas.addEventListener('mousemove', (event)=>  { event.preventDefault() })
    window.addEventListener('touchend', () => { this.endGesture() })
    window.addEventListener('mouseup', () => { this.endGesture() })

    if (this.isAndroid) {
      window.addEventListener("deviceorientation", this.handleOrientation)
    } else {
      window.addEventListener("devicemotion", this.handleMotion)
    }

    window.addEventListener('orientationchange', () => {
      this.motionInitial.x = 0
      this.motionInitial.y = 0
    })

  },
  methods: {
    loadLayers() {
      let loadCounter = 0

      this.loadedLayers = this.layers;

      this.loadedLayers.sort((a, b) => {
        return a.zIndex - b.zIndex;
      })

      this.loadedLayers.forEach((layer) => {

        if (!layer.initialPosition) {
          layer.initialPosition = {x: 0, y: 0}
        }

        if (!layer.opacity && layer.opacity !== 0) {
          layer.opacity = 1
        }

        if (!layer.scale) {
          layer.scale = 1
        }

        layer.image = new Image()
        layer.image.onload = () => {
          loadCounter += 1
          if (loadCounter < this.loadedLayers.length) return
          requestAnimationFrame(this.drawCanvas)
        }
        layer.image.src = layer.src
      })
    },
    drawCanvas() {
      this.context.clearRect(0,0, this.canvas.width, this.canvas.height);

      const coef = this.isAndroid ? 0.5 : 1.2
      let rotateX = (this.pointer.y * -0.15) + (this.motion.y * -coef)
      if (Math.abs(rotateX) > this.limitAngleY) {
        rotateX = this.limitAngleY * Math.sign(rotateX)
      }
      let rotateY = (this.pointer.x * 0.15) + (this.motion.x * coef)
      if (Math.abs(rotateY) > this.limitAngleX) {
        rotateY = this.limitAngleX * Math.sign(rotateY)
      }
      this.canvas.style.transform = `rotateX(${rotateX}deg) rotateY(${rotateY}deg)`

      const paddingScalingX = 1 / ( 1 + (this.paddingX * 2 / this.width));
      const paddingScalingY = 1 / ( 1 + (this.paddingY * 2 / this.height));

      this.context.setTransform(paddingScalingX,0,0,paddingScalingY,this.paddingX * paddingScalingX,this.paddingY * paddingScalingY)

      this.loadedLayers.forEach((layer) => {
        layer.position = this.getOffset(layer)
        this.context.globalCompositeOperation = layer.blend || 'source-over'
        this.context.globalAlpha = layer.opacity
        this.context.drawImage(layer.image, layer.initialPosition.x + layer.position.x, layer.initialPosition.y + layer.position.y, layer.image.width * layer.scale, layer.image.height * layer.scale)
      })

      this.context.resetTransform();


      requestAnimationFrame(this.drawCanvas)
    },
    getOffset(layer) {

      let movementX = this.pointer.x + this.motion.x
      let movementY = this.pointer.y + this.motion.y

      if (Math.abs(movementX) > Math.abs(this.maxMovementX)) {
        movementX = this.maxMovementX * Math.sign(movementX)
      }

      if (Math.abs(movementY) > Math.abs(this.maxMovementY)) {
        movementY = this.maxMovementY * Math.sign(movementY)
      }

      const offsetX = movementX * layer.zIndex
      const offsetY = movementY * layer.zIndex

      return {
        x: offsetX,
        y: offsetY,
      }
    },
    pointerStart(event) {
      this.isMoving = true
      if (event.type === 'touchstart') {
        this.pointerInitial.x = event.touches[0].clientX
        this.pointerInitial.y = event.touches[0].clientY
      } else if (event.type === 'mousedown') {
        this.pointerInitial.x = event.clientX
        this.pointerInitial.y = event.clientY
      }
    },
    pointerMove(event) {
      if (!this.isMoving) return
      let currentX = 0
      let currentY = 0
      if (event.type === 'touchmove') {
        currentX = event.touches[0].clientX
        currentY = event.touches[0].clientY
      } else if (event.type === 'mousemove') {
        currentX = event.clientX
        currentY = event.clientY
      }
      let diffX = currentX - this.pointerInitial.x
      let diffY = currentY - this.pointerInitial.y

      const touchMultiplier = 0.3

      diffX = diffX * touchMultiplier
      diffY = diffY * touchMultiplier

      this.pointer.x = diffX
      this.pointer.y = diffY
    },
    endGesture() {
      this.isMoving = false
      // Tween back to 0
      this.pointer = { x:0, y:0 }
    },
    handleMotion(event) {

      const alpha = event.rotationRate.alpha
      const beta = event.rotationRate.beta

      this.totalX += beta
      this.totalY += alpha

      if (Math.abs(this.totalX) > this.maxOffset) {
        this.totalX = this.maxOffset * Math.sign(this.totalX)
      }
      if (Math.abs(this.totalY) > this.maxOffset) {
        this.totalY = this.maxOffset * Math.sign(this.totalY)
      }

      let xOffset = - this.totalY / 100
      let yOffset = this.totalX / 100

      const motionMultiplier = this.isAndroid ? 0.5 : 2

      xOffset = xOffset * motionMultiplier
      yOffset = yOffset * motionMultiplier

      this.motion.x = xOffset
      this.motion.y = yOffset

      if (screen.orientation.angle === 90) {
        this.motion.x = -xOffset
        this.motion.y = -yOffset
      } else if (screen.orientation.angle === -90) {
        this.motion.x = xOffset
        this.motion.y = yOffset
      } else if (screen.orientation.angle === 180) {
        this.motion.x = -yOffset
        this.motion.y = xOffset
      } else if (screen.orientation.angle === 0) {
        this.motion.x = yOffset
        this.motion.y = -xOffset
      }
    },
    handleOrientation(event) {
      if (!this.motionInitial.x && !this.motionInitial.y) {
        this.motionInitial.x = event.beta
        this.motionInitial.y = event.gamma
      }

      if (screen.orientation.angle === 0) {
        this.motion.x = event.gamma - this.motionInitial.y
        this.motion.y = event.beta - this.motionInitial.x
      } else if (screen.orientation.angle === 90) {
        this.motion.x = event.beta - this.motionInitial.x
        this.motion.y = -event.gamma + this.motionInitial.y
      } else if (screen.orientation.angle === -90) {
        this.motion.x = -event.beta + this.motionInitial.x
        this.motion.y = event.gamma - this.motionInitial.y
      } else {
        this.motion.x = -event.gamma + this.motionInitial.y
        this.motion.y = -event.beta + this.motionInitial.x
      }
    },
  },
  watch: {
    layers() {
      this.loadLayers();
    },
  },
}
</script>

<style scoped>
.parallax-canvas {
  width: 100%;
  height: auto;
  position: relative;
  backface-visibility: hidden;
}
</style>