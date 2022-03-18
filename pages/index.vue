<template>
  <div class="page home">
    <div
      class="gallery"
      @mouseenter="handleMouseEnter"
      @mouseleave="handleMouseLeave"
      ref="gallery"
    >
      <div
        class="gallery-image"
        v-for="(img, index) in images"
        :key="`image-${index}`"
        ref="galleryImage"
        :style="{
          height: `${img.height}px`,
          width: `${img.width}px`,
        }"
      >
        <img
          :src="img.src"
        />
      </div>
    </div>
  </div>
</template>

<script>
import interact from "interactjs"
export default {
  name: 'home',
  data() {
    return {
      distance: 0,
      dragData: {
        initial: 0,
        offset: 0,
      },
      images: [],
      interval: null,
      isDragged: false,
      marginValue: 10,
      numberOfImages: 11,
      totalWidth: 0,
    }
  },
  computed: {
    parentHeight() {
      return this.$refs.gallery.getBoundingClientRect().height
    },
    parentWidth() {
      return this.$refs.gallery.getBoundingClientRect().width
    },
  },
  mounted() {
    this.interval = setInterval(this.handleInfinite, 16)

    for (let i = 0; i < this.numberOfImages; i++) {
      const img = document.createElement('img')
      img.onload = () => {
        const width = img.height / img.width
        this.images.push({
          height: this.parentHeight,
          width: this.parentHeight * img.width / img.height,
          src: img.src,
          y: '-50%',
        })
        this.totalWidth += this.marginValue + (this.parentHeight * img.width / img.height)
      }
      img.src = `https://picsum.photos/600/400?random=${i}`
    }

    let draggableEl = this.$refs.gallery
    this.initInteract(draggableEl)
  },
  methods: {
    handleDragMove(e) {
      this.distance += -e.dx
    },
    handleInfinite() {
      this.distance += 1
    },
    handleMouseEnter() {
      clearInterval(this.interval)
    },
    handleMouseLeave() {
      this.interval = setInterval(this.handleInfinite, 16)
    },
    initInteract(selector) {
      interact(selector).draggable({
        // enable inertial throwing
        inertia: true,
        // keep the element within the area of it's parent
        // restrict: {
        //   restriction: "parent",
        //   endOnly: true,
        //   elementRect: { top: 0, left: 0, bottom: 1, right: 1 }
        // },
        // enable autoScroll
        autoScroll: true,

        // call this function on every dragmove event
        onmove: this.handleDragMove,
        // call this function on every dragend event
        // onend: this.handleDragEnd
      })
    }
  },
  watch: {
    distance(val) {
      for (let i = 0; i < this.images.length; i++) {
        // loop right
        if (
          Math.abs(this.images[i].x - val) > this.totalWidth - this.parentWidth &&
          this.images[i].x + this.images[i].width < val
        ) {
          this.images[i].x = this.totalWidth + this.images[i].x
        }

        // loop left
        if (
          Math.abs(this.images[i].x - val) > this.totalWidth - this.parentWidth &&
          this.images[i].x > val
        ) {
          this.images[i].x = this.images[i].x - this.totalWidth
        }

        // add translate if within screen
        if (
          this.images[i].x < val + this.parentWidth &&
          this.images[i].x + this.images[i].width > val
        ) {
          this.images[i].translate = this.images[i].x - val
          this.$refs.galleryImage[i].classList.add('visible')
        } else {
          this.images[i].translate = -9999
          this.$refs.galleryImage[i].classList.remove('visible')
        }

        this.$refs.galleryImage[i].style.transform = `translate3d(${this.images[i].translate}px, ${this.images[i].y}, 0)`
      }
    },
    images(val) {
      if (val.length !== this.numberOfImages) {
        return false
      }
      for (let i = 0; i < val.length; i++) {
        let x = 0
        for (let j = 0; j < i; j++) {
          x += this.marginValue + this.images[j].width
        }
        this.images[i] = {...this.images[i], x, translate: x}
      }
    }
  }
}
</script>

<style lang="scss">
body {
  margin: 0;
}
.home {
  width: 100vw;
  height: 100vh;
  display: flex;
  align-items: center;
  .gallery {
    width: 100%;
    height: 500px;
    position: relative;
    overflow: hidden;

    .gallery-image {
      position: absolute;
      top: 50%;
      left: 0;

      &.visible {
        img {
          opacity: 1;
          transform: translateY(0);
        }
      }
    }

    img {
      width: 100%;
      height: 100%;
      opacity: 0;
      transform: translateY(20px);
      display: block;
      transition: all 0.4s ease-out;
    }
  }
}
</style>