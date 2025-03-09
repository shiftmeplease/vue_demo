<template>
  <div class="_tooltip" :style="{ top: `${y}px`, left: `${x}px` }">
    <slot></slot>
  </div>
</template>

<script setup>
const props = defineProps({
  x: Number,
  y: Number,
  text: String,
});
</script>

<style scoped>
._tooltip {
  position: absolute;
  z-index: 999;
  background-color: rgba(0, 0, 0, 0.75);
  color: white;
  padding: 5px;
  border-radius: 3px;
  pointer-events: none;
  transform: translate(-50%, -100%);
  white-space: nowrap;
}
</style>