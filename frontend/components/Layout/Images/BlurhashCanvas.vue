<template>
  <v-fade-transition>
    <canvas
      ref="canvas"
      :key="`canvas-${hash}`"
      :width="width"
      :height="height"
      class="absolute"
    />
  </v-fade-transition>
</template>

<script lang="ts">
import Vue from 'vue';
import getPixels from '~/plugins/workers/blurhash.worker';

export default Vue.extend({
  props: {
    hash: {
      type: String,
      required: true
    },
    width: {
      type: Number,
      default: 32
    },
    height: {
      type: Number,
      default: 32
    },
    punch: {
      type: Number,
      default: 1
    }
  },
  data() {
    return {
      pixels: undefined as Uint8ClampedArray | undefined
    };
  },
  watch: {
    hash(): void {
      this.$nextTick(() => {
        this.getPixels();
      });
    },
    pixels(): void {
      this.draw();
    }
  },
  mounted() {
    this.getPixels();
  },
  methods: {
    draw(): void {
      const ctx = (this.$refs.canvas as HTMLCanvasElement).getContext('2d');
      const imageData = ctx?.createImageData(this.width, this.height);

      if (imageData && this.pixels) {
        imageData.data.set(this.pixels);
        ctx?.putImageData(imageData, 0, 0);
      }
    },
    async getPixels(): Promise<void> {
      try {
        this.pixels = await getPixels(
          this.hash,
          this.width,
          this.height,
          this.punch
        );
      } catch {
        this.pixels = undefined;
        this.$emit('error');
      }
    }
  }
});
</script>

<style lang="scss" scoped>
.fade-fast-enter-active,
.fade-fast-leave-active {
  transition: opacity 0.15s;
}

.fade-fast-enter,
.fade-fast-leave-to {
  opacity: 0;
}

.absolute {
  height: 100%;
  width: 100%;
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
}
</style>
