<template>
  <g :transform="transform" class="comp">
    <rect class="backdrop" x="0" y="0" :width="width" :height="height" />
    <rect
      @pointerdown="onPointerDown"
      @pointermove="onPointerMove"
      @pointerup="onPointerUp"
      :x="handle"
      y="0"
      width="20"
      :height="height"
      fill="#333"
      rx="2"
      ry="2"
    />
  </g>
</template>

<script>
function lerp(a, b, t) {
  return a + (b - a) * t;
}

function clamp(x, min, max) {
  if (x < min) return min;
  else if (x > max) return max;
  return x;
}

function hypot(x, y) {
  return Math.sqrt(x * x + y * y);
}

export default {
  props: ["min", "max", "start", "end", "x", "y", "height", "width"],
  data() {
    return {
      offset: null,
      old: {}
    };
  },
  methods: {
    onPointerDown(e) {
      e.target.setPointerCapture(e.pointerId);
      this.offset = {
        x: e.offsetX,
        y: e.offsetY
      };
      this.old = {
        start: this.start
      };
    },
    onPointerMove(e) {
      if (this.offset) {
        const diff = e.offsetX - this.offset.x;
        const start = (diff * this.length) / this.width + this.old.start;
        this.$emit("move", start);
      }
    },
    onPointerUp() {
      this.offset = null;
    }
  },
  computed: {
    transform() {
      return `translate(${this.x}, ${this.y})`;
    },
    length() {
      return this.max - this.min;
    },
    handle() {
      return ((this.start - this.min) / this.length) * this.width;
    }
  }
};
</script>

<style>
.comp > .backdrop {
  transition: all 0.2s;
  fill: rgba(255, 255, 255, 0);
}
.comp:hover > .backdrop {
  fill: rgba(255, 255, 255, 0.3);
}
</style>