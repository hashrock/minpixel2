<template>
  <div>
    <svg width="400" height="400" ref="canv" preserveAspectRatio="none">
      <g :transform="transformString">
        <image
          @pointermove="onPointerMove"
          @pointerdown="onPointerDown"
          @pointerup="onPointerUp"
          :href="imageData"
          x="0"
          y="0"
          width="200"
          height="200"
        />
      </g>
      <Scroller
        :min="-500"
        :max="500"
        :start="view.x"
        :end="30"
        :x="0"
        :y="400-15"
        :width="400"
        :height="15"
        @move="onMoveHScroll"
      />
    </svg>
    <Pallete :selected-color="selectedColor" @select="onSelectColor"></Pallete>

    <p>
      <input type="range" v-model.number="view.x" min="-500" max="500" />
      <input type="range" v-model.number="view.y" min="-500" max="500" />
      <input type="range" v-model.number="view.scale" min="0" max="10" step="0.1" />
    </p>
    <p v-if="offset">{{offset.x}} {{offset.y}}</p>
  </div>
</template>
<style>
svg {
  background: #555;
  display: block;
}
image {
  image-rendering: pixelated;
  image-rendering: -moz-crisp-edges;
  transform: scale(4);
}
body {
  margin: 2rem;
  background: #333;
}
</style>
<script>
import Pallete from "./Pallete.vue";
import Scroller from "./Scroller.vue";

const canvas = document.createElement("canvas");
canvas.width = 200;
canvas.height = 200;
const ctx = canvas.getContext("2d");
const arr = new Uint8ClampedArray(32 * 32 * 4);
function screenToSvg(point, el, svg, floor) {
  const pt = svg.createSVGPoint();
  pt.x = point.x;
  pt.y = point.y;
  const p = pt.matrixTransform(el.getScreenCTM().inverse());
  return {
    x: floor ? Math.floor(p.x) : p.x,
    y: floor ? Math.floor(p.y) : p.y
  };
}

async function getImageData() {
  const blob = await new Promise(resolve => {
    canvas.toBlob(resolve, "image/jpeg");
  });

  return new Uint8Array(await blob.arrayBuffer());
}

export default {
  data() {
    return {
      imageData: "",
      offset: null,
      view: {
        x: 10,
        y: 10,
        scale: 2
      },
      selectedColor: "#000000"
    };
  },
  computed: {
    transformString() {
      return `translate(${-1 * this.view.x}, ${-1 * this.view.y}) scale(${
        this.view.scale
      })`;
    }
  },
  methods: {
    line(x0, y0, x1, y1) {
      const dx = Math.abs(x1 - x0);
      const dy = Math.abs(y1 - y0);
      const sx = x0 < x1 ? 1 : -1;
      const sy = y0 < y1 ? 1 : -1;
      let err = dx - dy;

      while (true) {
        this.setPixel(x0, y0);
        if (x0 == x1 && y0 == y1) {
          break;
        }
        const e2 = err << 1;
        if (e2 > -dy) {
          err -= dy;
          x0 += sx;
        }
        if (e2 < dx) {
          err += dx;
          y0 += sy;
        }
      }
    },
    setPixel(x, y) {
      if (x < 0) {
        return;
      }
      if (y < 0) {
        return;
      }
      if (x > 31) {
        return;
      }
      if (y > 31) {
        return;
      }

      const start = (y * 32 + x) * 4;
      arr[start] = parseInt(this.selectedColor.slice(1, 3), 16);
      arr[start + 1] = parseInt(this.selectedColor.slice(3, 5), 16);
      arr[start + 2] = parseInt(this.selectedColor.slice(5, 7), 16);
      arr[start + 3] = 255;
    },
    onPointerDown(e) {
      const rect = e.target;
      this.offset = screenToSvg(
        { x: e.clientX, y: e.clientY },
        rect,
        this.$refs.canv,
        true
      );
      rect.setPointerCapture(e.pointerId);
    },
    onPointerMove(e) {
      if (this.offset) {
        const rect = e.target;
        const p = screenToSvg(
          { x: e.clientX, y: e.clientY },
          rect,
          this.$refs.canv,
          true
        );
        this.line(this.offset.x, this.offset.y, p.x, p.y);
        this.offset = p;
        this.redraw();
      }
    },
    onPointerUp(e) {
      this.offset = null;
    },
    redraw() {
      let imageData = new ImageData(arr, 32);
      ctx.putImageData(imageData, 0, 0);
      this.imageData = canvas.toDataURL();
    },
    onSelectColor(color) {
      this.selectedColor = color;
    },
    onMoveHScroll(value) {
      this.view.x = value;
    }
  },
  mounted() {
    for (let i = 0; i < arr.length; i += 4) {
      const p = Math.floor(Math.random() * 255);
      arr[i + 0] = 255;
      arr[i + 1] = 255;
      arr[i + 2] = 255;
      arr[i + 3] = 255;
    }
    this.redraw();
  },
  components: {
    Pallete,
    Scroller
  }
};
</script>