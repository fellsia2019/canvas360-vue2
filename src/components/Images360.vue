<template>
  <div
    v-if="isExistsComponentData"
    class="images-360"
    @mousedown="start"
    @touchstart="start"
  >
    <canvas
      ref="canvas"
      class="images-360__canvas"
      width="400"
      height="400"
    />
  </div>
</template>

<script>
export default {
  props: {
    images: {
      type: Array,
      default: () => [],
    },
  },
  data() {
    return {
      activeIndex: 0,
      startPosition: 0,
      prevMovePosition: 0,
      prevTo: 'right',
      maxDelta: 0,
      range: 0,
      currentDelta: 0,
      maxIndex: 0,
      minIndex: 0,
      ctx: null,
      imagesList: [],
      newWidthImg: 0,
      newHeightImg: 0,
      prevWindowWidth: 0,
    };
  },
  computed: {
    isExistsComponentData() {
      return this.images && this.images.length;
    },
  },
  mounted() {
    window.addEventListener('mouseup', this.end);
    window.addEventListener('mousemove', this.move);
    window.addEventListener('resize', this.reinit);

    window.addEventListener('touchend', this.end);
    window.addEventListener('touchmove', this.move);

    this.$nextTick(() => {
      this.initSettings();
    });
  },
  beforeDestroy() {
    window.removeEventListener('mouseup', this.end);
    window.removeEventListener('mousemove', this.move);
    window.removeEventListener('resize', this.reinit);
  },
  methods: {
    initSettings() {
      this.$nextTick(() => {
        this.canvas = this.$refs.canvas;
        this.ctx = this.canvas ? this.canvas.getContext('2d') : null;
        if (!this.ctx) return;
        this.ctx.imageSmoothingEnabled = true;
        this.maxDelta = this.$el.getBoundingClientRect().width;
        this.range = (this.maxDelta / this.images.length);
        this.maxIndex = this.images.length - 1;
        this.minIndex = 0;
        this.prevWindowWidth = window.innerWidth;
        this.containerWidth = this.$el.getBoundingClientRect().width * window.devicePixelRatio;
        this.containerHeight = this.$el.getBoundingClientRect().height * window.devicePixelRatio;

        this.createImageList();
      });
    },
    createImageList() {
      let isLoadedImg = 0;
      this.imagesList = [];

      this.images.forEach((img) => {
        const image = new Image();
        image.src = img.large;
        image.srcset = this.getSrcset(img);
        image.onload = () => {
          const aspectRatio = image.width / image.height;

          if (this.containerHeight * aspectRatio <= this.containerWidth) {
            this.newWidthImg = this.containerHeight * aspectRatio;
            this.newHeightImg = this.containerHeight;
          } else {
            this.newWidthImg = this.containerWidth;
            this.newHeightImg = this.containerWidth / aspectRatio;
          }

          this.canvas.width = this.containerWidth;
          this.canvas.height = this.containerHeight;

          isLoadedImg += 1;

          if (isLoadedImg >= this.imagesList.length) {
            this.drawFrame();
          }
        };

        this.imagesList = [...this.imagesList, image];
      });
    },
    drawFrame() {
      this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
      const x = (this.containerWidth - this.newWidthImg) / 2;
      const y = (this.containerHeight - this.newHeightImg) / 2;

      this.ctx.drawImage(this.imagesList[this.activeIndex], x, y, this.newWidthImg, this.newHeightImg);
      requestAnimationFrame(this.drawFrame);
    },
    getSrcset(img) {
      return `${img.default} 1x, ${img.large} 2x`;
    },
    start(e) {
      this.canMove = true;
      this.startPosition = e.touches ? e.touches[0].pageX : e.pageX;
    },
    end() {
      this.canMove = false;
    },
    move(e) {
      if (!this.canMove) return;
      const pageX = e.touches ? e.touches[0].pageX : e.pageX;
      const delta = pageX - this.startPosition;

      let to;
      if (pageX > this.prevMovePosition) {
        to = 'right';
      } else if (pageX < this.prevMovePosition) {
        to = 'left';
      } else {
        to = this.prevTo;
      }

      this.prevTo = to;
      this.prevMovePosition = pageX;
      this.currentDelta = delta;

      this.setActive(to, pageX);
    },
    setActive(to = this.prevTo, prevMovePosition) {
      let nextIndex = this.activeIndex;
      if (to === 'right') {
        if (this.currentDelta > this.range) {
          if (nextIndex + 1 >= this.maxIndex) {
            nextIndex = 0;
          } else {
            nextIndex += 1;
          }
          this.startPosition = prevMovePosition;
        }
      }

      if (to === 'left') {
        if ((this.currentDelta * -1) > this.range) {
          if (nextIndex - 1 <= this.minIndex) {
            nextIndex = this.maxIndex;
          } else {
            nextIndex -= 1;
          }
          this.startPosition = prevMovePosition;
        }
      }

      this.activeIndex = nextIndex;
    },
    reinit() {
      if (this.prevWindowWidth === window.innerWidth) return;
      this.initSettings();
    },
  },
};

</script>

<style lang="scss">

$b: '.images-360';

#{$b} {
  position: relative;
  width: 100%;
  height: 100%;
  border: 1px solid fuchsia;

  // .images-360__canvas
  &__canvas {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;

    // .images-360__canvas-img
    &-img {
      width: 100%;
      height: 100%;
      object-fit: contain;
      object-position: center;
    }
  }
}
</style>