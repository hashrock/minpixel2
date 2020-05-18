<template>
  <svg width="400" height="40">
    <g transform="translate(20,20)">
      <circle
        v-for="(color, idx) in colors"
        :key="idx"
        :cx="24*idx"
        :cy="0"
        :r="8"
        :fill="color"
        :stroke="selectedColor===color ? `#6666FF` : `rgba(80,80,80,0.5)`"
        stroke-width="6"
        paint-order="stroke fill"
        @click="onSelect(color)"
      />
    </g>
  </svg>
</template>

<script>
const colors = [
  "#000000",
  "#9d9d9d",
  "#ffffff",
  "#be2633",
  "#e06f8b",
  "#493c2b",
  "#a46422",
  "#eb8931",
  "#f7e26b",
  "#2f484e",
  "#44891a",
  "#a3ce27",
  "#1b2632",
  "#005784",
  "#31a2f2",
  "#b2dcef"
];
export default {
  props: ["selectedColor"],
  data() {
    return {
      colors
    };
  },
  methods: {
    onSelect(color) {
      this.$emit("select", color);
    }
  }
};
</script>

<style>
</style>