<template>
  <div class="container">
    <div class="withBorder">
      <CanvasParallax :layers="layers" width="500" height="500" :padding-x="100" :padding-y="100"></CanvasParallax>
    </div>
  </div>
</template>

<script>
import CanvasParallax from "@/components/CanvasParallax";

export default {
  name: 'App',
  components: {
    CanvasParallax,
  },
  data() {
    return {
      layers: [
        {
          src: './logo.png',
          zIndex: 0,
          initialPosition: { x: 130, y: 10 },
          scale: 1.2,
        },
        {
          src: './logo.png',
          zIndex: 0,
          initialPosition: { x: 10, y: 250 },
          scale: 1.2,
        },
        {
          src: './logo.png',
          zIndex: 0,
          initialPosition: { x: 250, y: 250 },
          scale: 1.2,
        },
        {
          src: './logo.png',
          zIndex: 0.5,
          initialPosition: { x: 10, y: 10 },
          opacity: 0.6,
        },
        {
          src: './logo.png',
          zIndex: 0.5,
          initialPosition: { x: 290, y: 10 },
          opacity: 0.6,
        },
        {
          src: './logo.png',
          zIndex: 0.5,
          initialPosition: { x: 290, y: 290 },
          opacity: 0.6,
        },
        {
          src: './logo.png',
          zIndex: 0.5,
          initialPosition: { x: 10, y: 290 },
          opacity: 0.6,
        },
        {
          src: './logo.png',
          zIndex: 0.8,
          initialPosition: { x: 100, y: 100 },
          scale: 1.5,
          opacity: 0.8,
        },
        {
          src: './logo.png',
          zIndex: 0.8,
          initialPosition: { x: 35 , y: 150 },
          scale: 0.5,
          opacity: 0.3,
        },
        {
          src: './logo.png',
          zIndex: 1,
          initialPosition: { x: 220, y: 400 },
          scale: 0.5,
          opacity: 0.3,
        },
      ],
    };
  },
}
</script>

<style>
.container {
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
}

.withBorder {
  border: black 2px solid;
  border-radius: 4px;
  /*padding: 10px;*/
}
</style>
