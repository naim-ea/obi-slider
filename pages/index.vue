<template>
  <div class="page home">
    <!-- gallery div is the outer div it -->
    <div
      class="gallery"
      @mouseenter="handleMouseEnter"
      @mouseleave="handleMouseLeave"
      ref="gallery"
    >
      <!-- we create a div for each image needed with dynamic width and height because they're absolutely positionned -->
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
// Import interactjs for drag action
import interact from "interactjs"

export default {
  name: 'home',
  data() {
    return {
      // distance is the distance travelled in pixels
      distance: 0,

      // images is an array which contains data about all the images
      images: [],

      // interval is a variable containing the interval for automatic sliding
      interval: null,

      // marginValue is the value in pixels of space between each image in carousel
      marginValue: 10,

      // numberOfImages is the number of images needed
      numberOfImages: 11,

      // totalWidth is the full length of all the photos next to each other
      totalWidth: 0,
    }
  },
  computed: {
    parentHeight() {
      // Height of 'gallery' div
      return this.$refs.gallery.getBoundingClientRect().height
    },
    parentWidth() {
      // Width of 'gallery' div
      return this.$refs.gallery.getBoundingClientRect().width
    },
  },
  mounted() {
    // Init interval for automatic sliding
    this.interval = setInterval(this.handleInfinite, 16)

    let numberOfImagesLoaded = 0

    // Load all images and add data about them
    for (let i = 0; i < this.numberOfImages; i++) {
      const img = document.createElement('img')

      img.onload = () => {
        // Calculate height of smaller images
        const height = numberOfImagesLoaded % 2 === 0 ? this.parentHeight : this.parentHeight * 0.86
        const width = height * img.width / img.height

        this.images.push({
          // height gets the height of the 'gallery' div because it expands to full height
          height: height,

          // width is a ratio between the parentHeight and the ratio of the original image (to fully keep the image)
          width: width,

          src: img.src,

          // y = -50% because we need them to be centered vertically
          y: '-50%',
        })

        // we calculate the totalwidth of the slider within this loop
        this.totalWidth += width + this.marginValue

        numberOfImagesLoaded++
      }

      // we keep the srcs here to display it in the template
      img.src = `https://picsum.photos/600/400?random=${i}`
    }

    // Init inertia drag action on 'gallery' div
    let draggableEl = this.$refs.gallery
    this.initInteract(draggableEl)
  },
  methods: {
    handleDragMove(e) {
      // increment distance variable with the dx variable from the drag event
      // you can maybe change value incremented here to make drag action smoother
      this.distance += -e.dx
    },
    handleInfinite() {
      // increment distance automatically within interval
      this.distance += 1
    },
    handleMouseEnter() {
      // clear interval so that slider stops automatically when the user enters the 'gallery' div with its mouse
      clearInterval(this.interval)
    },
    handleMouseLeave() {
      // re init interval when user leaves slider with its mouse
      this.interval = setInterval(this.handleInfinite, 16)
    },
    initInteract(selector) {
      // init interactjs
      interact(selector).draggable({
        // enable inertial throwing
        inertia: {
          // to make drag and drop smoother, change values here depending on these : https://interactjs.io/docs/inertia/
          resistance: 4,
        },

        // call this function on every dragmove event
        onmove: this.handleDragMove,
      })
    }
  },
  watch: {
    distance(val) {
      // when distance variable changes, move the images according to if their x value is in the screen or not
      for (let i = 0; i < this.images.length; i++) {
        // loop right
        if (
          Math.abs(this.images[i].x - val) > this.totalWidth - this.parentWidth &&
          this.images[i].x + this.images[i].width < val
        ) {
          // change image x to match the distance that is above the totalWidth
          this.images[i].x = this.totalWidth + this.images[i].x
        }

        // loop left
        if (
          Math.abs(this.images[i].x - val) > this.totalWidth - this.parentWidth &&
          this.images[i].x > val
        ) {
          // change image x to match the distance that is below the totalWidth
          this.images[i].x = this.images[i].x - this.totalWidth
        }

        // add translate if within screen
        if (
          this.images[i].x < val + this.parentWidth &&
          this.images[i].x + this.images[i].width > val
        ) {
          // if within screen, add translate value and class name
          this.images[i].translate = this.images[i].x - val
          this.$refs.galleryImage[i].classList.add('visible')
        } else {
          // if within screen, remove translate value and class name
          this.images[i].translate = -9999
          this.$refs.galleryImage[i].classList.remove('visible')
        }

        // add transform style
        this.$refs.galleryImage[i].style.transform = `translate3d(${this.images[i].translate}px, ${this.images[i].y}, 0)`
      }
    },
    images(val) {
      // when images array is populated (= when all images are loaded), check if it matches the initial number of images
      if (val.length !== this.numberOfImages) {
        return false
      }
      // if all images are loaded, populate each image with x value, so calculations purposes when moving the slider
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
      transform: translateY(50px);
      display: block;
      transition: all 0.4s ease-out;
    }
  }
}
</style>