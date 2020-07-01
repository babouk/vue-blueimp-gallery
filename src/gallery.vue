<template>
  <div :id="id" class="blueimp-gallery blueimp-gallery-controls" :class="{ 'blueimp-gallery-carousel': carousel }">
    <div class="slides"></div>
    <h1 class="title"></h1>
    <div class="caption"></div>
    <a class="prev">
      <slot name="prev">‹</slot>
    </a>
    <a class="next">
      <slot name="next">›</slot>
    </a>
    <a v-if="!carousel" class="close">
      <slot name="close">×</slot>
    </a>
    <ol v-if="!carousel" class="indicator"></ol>
    <a v-if="carousel" class="play-pause"></a>
  </div>
</template>

<script>
import "blueimp-gallery/css/blueimp-gallery.min.css";
import "blueimp-gallery/js/blueimp-gallery-fullscreen.js";
import "blueimp-gallery/js/blueimp-gallery-video.js";
import "blueimp-gallery/js/blueimp-gallery-youtube.js";
import blueimpGallery from "blueimp-gallery/js/blueimp-gallery.js";

export default {
  name: "BlueimpGallery",
  props: {
    carousel: {
      type: Boolean,
      default: false,
    },
    id: {
      type: String,
      default: "blueimp-gallery",
    },
    images: {
      type: Array,
      default: () => [],
    },
    options: {
      type: Object,
      default: () => ({}),
    },
    index: {
      type: Number,
      default: -1,
    },
  },
  data() {
    return {
      instance: null,
    };
  },
  destroyed() {
    if (this.instance !== null) {
      this.instance.destroyEventListeners();
      this.instance.close();
      this.instance = null;
    }
  },
  watch: {
    index(value) {
      if (this.carousel) {
        return;
      }

      if (value !== null) {
        if (this.instance == null) {
          this.instantiate();
        }
        this.instance.slide(value);
      } else {
        if (this.instance !== null) {
          this.instance.close();
        }
        this.$emit("close");
      }
    },
  },
  methods: {
    instantiate() {
      const options = Object.assign(
        {
          toggleControlsOnReturn: false,
          toggleControlsOnSlideClick: false,
          closeOnSlideClick: false,
          carousel: this.carousel,
          container: `#${this.id}`,
          index: this.index,
          onopen: () => this.$emit("onopen"),
          onopened: () => this.$emit("onopened"),
          onslide: this.onSlide,
          onslideend: (index, slide) => this.$emit("onslideend", { index, slide }),
          onslidecomplete: (index, slide) => this.$emit("onslidecomplete", { index, slide }),
          onclose: () => this.$emit("close"),
          onclosed: () => this.$emit("onclosed"),
        },
        this.options
      );

      if (this.carousel) {
        options.container = this.$el;
      }

      this.instance = blueimpGallery(this.images, options);
    },

    onSlide(index, slide) {
      this.$emit("onslide", { index, slide });
      const image = this.images[index];
      if (image !== undefined) {
        const caption = image.caption;
        const node = this.instance.container.find(".caption");
        if (caption !== undefined) {
          node[0].innerHTML = caption;
          node[0].style.display = "block";
        } else {
          node[0].innerHTML = "";
          node[0].style.display = "none";
        }
      }
    },
  },
};
</script>
<style>
.blueimp-gallery > .caption {
  background: black;
  opacity: 0.7;
  bottom: 0;
  color: #fff;
  display: none;
  left: 50%;
  position: absolute;
  text-align: center;
  transform: translate(-50%, 0%);
  width: 100%;
}
.blueimp-gallery-controls > .caption {
  display: block;
}
</style>
