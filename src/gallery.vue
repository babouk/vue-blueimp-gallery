<template>
  <div :id="id" class="blueimp-gallery blueimp-gallery-controls" :class="{ 'blueimp-gallery-carousel': carousel }">
    <div class="slides"></div>
    <h1 class="title"></h1>
    <div class="caption-panel">
      <a class="control button is-primary caption-edit" @click="toggleCaptionInput()">
        <slot name="caption-edit">Edit</slot>
      </a>
      <div v-if="!showCaptionInput && caption.html !== undefined && caption.html.length > 0" class="caption" v-html="caption.html"></div>
      <textarea ref="captionInput" v-if="showCaptionInput" class="caption-textarea" rows="5" v-model="caption.markdown"></textarea>
    </div>
    <a class="prev">
      <slot name="prev">‹</slot>
    </a>
    <a class="next">
      <slot name="next">›</slot>
    </a>
    <div class="toolbar">
      <slot name="toolbar">
        <a class="control button remove is-primary" @click="removeActiveItem()">
          Delete
        </a>
        <a v-if="!carousel" class="control button close is-primary">
          ×
        </a>
      </slot>
    </div>
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
    items: {
      type: Array,
      default: () => [],
    },
    options: {
      type: Object,
      default: () => ({}),
    },
    origin: {
      type: Number,
      default: -1,
    },
  },
  data() {
    return {
      blimga: null, // blueimpGallery object
      showCaptionInput: false,
      caption: { markdown: "", html: "" },
      index: -1,
    };
  },
  destroyed() {
    if (this.blimga !== null) {
      this.blimga.destroyEventListeners();
      this.blimga.close();
      this.blimga = null;
    }
  },
  mounted() {},
  watch: {
    /**
     * Fire gallery set up when the origin property value changes.
     */
    origin(value) {
      if (this.carousel) {
        return;
      }

      if (value !== null && value >= 0) {
        this.setup();
        this.blimga.slide(value);
      } else {
        if (this.blimga !== null) {
          this.blimga.close();
        }
      }
    },
    /**
     * In case one or several items get deleted, reset the gallery with the new items
     * and position it at the previous slide.
     */
    items(newValue, oldValue) {
      if (newValue !== null && newValue.length > 0) {
        const diff = oldValue.length - newValue.length;
        if (diff > 0) {
          let position = this.blimga.getIndex();
          if (position > diff - 1) position = position - diff;
          else position = 0;
          this.setup(position);
        }
      }
    },
  },
  methods: {
    removeActiveItem() {
      const position = this.blimga.getIndex();
      if (this.options.onItemRemove !== undefined) {
        this.options.onItemRemove(position);
      }
    },
    onCaptionLoaded(caption) {
      this.caption = caption;
    },
    /**
     * Gallery current position.
     */
    getIndex() {
      return this.index;
    },
    setup(position) {
      const options = Object.assign(
        {
          toggleControlsOnEnter: false,
          toggleSlideshowOnSpace: false,
          toggleControlsOnSlideClick: false,
          closeOnSlideClick: false,
          carousel: this.carousel,
          container: `#${this.id}`,
          index: position !== undefined ? position : this.origin,
          onslide: (index, slide) => {
            if (this.items[index] !== undefined) {
              this.caption = { markdown: this.items[index].markdown, html: this.items[index].description };
            }
            this.index = index;
            this.showCaptionInput = false;
          },
        },
        this.options
      );

      if (this.carousel) {
        options.container = this.$el;
      }

      this.blimga = blueimpGallery(this.items, options);
    },
    async toggleCaptionInput() {
      if (this.showCaptionInput && this.options.onCaptionUpdate !== undefined) {
        const result = await this.options.onCaptionUpdate({ index: this.index, caption: { markdown: this.caption.markdown }, show: this.showCaptionInput });
        console.log("toggleCaption", result);
        const position = this.blimga.getIndex();
        if (this.items[position] !== undefined) {
          this.items[position].markdown = this.caption.markdown;
          this.items[position].description = result;
          this.caption.html = result;
        } else {
          console.log("[gallery.vue] [warning] no item found at position", position, this.items);
        }
      }
      this.showCaptionInput = !this.showCaptionInput;
    },
  },
};
</script>
<style>
.blueimp-gallery .toolbar {
  position: absolute;
  right: 1rem;
  top: 1rem;
}

.blueimp-gallery .caption-panel {
  background: black;
  bottom: 0;
  color: #fff;
  display: block;
  left: 50%;
  opacity: 0.7;
  position: absolute;
  text-align: center;
  transform: translate(-50%, 0%);
  width: 100%;
}

.blueimp-gallery .caption-panel .caption {
  width: 60%;
  margin: 0.5rem auto;
}

.blueimp-gallery .caption-panel .caption-edit {
  position: absolute;
  bottom: 1rem;
  right: 1rem;
}

.blueimp-gallery .caption-panel textarea {
  margin: 1rem;
  padding: 0.5rem;
  width: 40%;
}

.blueimp-gallery .control.button {
  margin: 0.3rem;
}
</style>
