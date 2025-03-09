<template>
  <g
    v-if="!element.type && element.children && element.children.length > 0"
    @click.stop="handleClick"
    :sectorId="elemId"
    @mousemove.self="handleMouseEnter"
    @mouseleave.stop="handleMouseLeave"
  >
    <!-- maybe not stop, if needed full cycle of event; so you need to implement a filter -->
    <svg-element
      v-for="(child, index) in element.children"
      :key="child.id || child.subId || index"
      :element="child"
      @mousemove="(ev)=>emitHover(child.id || child.subId,ev)"
      @leave="emitLeave"
      @click="emitClick"
    ></svg-element>
  </g>
  <path v-else-if="element.type === 'path'" v-bind="element.attributes"></path>
  <circle
    v-else-if="element.type === 'circle'"
    v-bind="element.attributes"
  ></circle>
  <rect v-else-if="element.type === 'rect'" v-bind="element.attributes"></rect>
  <text v-else-if="element.type === 'text'" v-bind="element.attributes">{{
    element.value
  }}</text>
  <line v-else-if="element.type === 'line'" v-bind="element.attributes"></line>
  <polyline
    v-else-if="element.type === 'polyline'"
    v-bind="element.attributes"
  ></polyline>
</template>
  
  <script setup>
import { computed } from "vue";

const emit = defineEmits(["hover", "leave", "click"]);

const props = defineProps({
  element: {
    type: Object,
    required: true,
  },
});

const element = props.element;

//TODO active

const elemId = computed(() => {
  if (element.id) return element.id;
  if (element.subId) return `sub-${element.subId}`;
  return "root";
});

const handleMouseEnter = (ev) => {
  emit("hover", elemId.value, ev);
};

const handleMouseLeave = () => {
  emit("leave", elemId.value);
};

const handleClick = () => {
  emit("click", elemId.value);
};

const emitHover = (id,ev) => {
  emit("hover", id, ev);
};

const emitLeave = (id) => {
  emit("leave", id);
};

const emitClick = (id) => {
  emit("click", id);
};
</script>