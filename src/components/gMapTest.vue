<template>
  <SvgTree
    :schemeData="schemeData"
    :width="intialSchemeData.sWidth"
    :height="intialSchemeData.sHeight"
    :styles="intialSchemeData.sStyles"
    @hover="handleHover"
    @leave="handleLeave"
    @click="handleClick"
  ></SvgTree>
</template>

<script setup>
import { computed, onMounted, reactive, watch, defineEmits, ref, onUnmounted  } from "vue";
import SvgTree from "@/components/MapComponents/SvgTree.vue";

const emit = defineEmits(["hover", "leave", "click"]);

const { schemeData, sectorToColor } = defineProps({
  schemeData: Array,
  sectorToColor: Object,
});

const handleHover = (elemId, event) => {
  emit("hover", elemId, event);
};

const handleLeave = (elemId) => {
  emit("leave", elemId);
};

const handleClick = (elemId) => {
  emit("click", elemId);
};

const intialSchemeData = computed(() => {
  if (!schemeData[0])
    return { sWidth: 100, sHeight: 100, sColorRegions: "", sStyles: "" };
  const styles = schemeData[0] ? atob(schemeData[0].sStyles) : "";
  return { ...schemeData[0], sStyles: styles };
});

const styleElement = ref(null);

const updateStyles = () => {
  if (
    styleElement.value &&
    sectorToColor &&
    Object.entries(sectorToColor).length > 0
  ) {
    const newStyles = Object.entries(sectorToColor)
      .map(([prop, val]) => {
        return `g[sectorId="${prop}"] path,
        g[sectorId="${prop}"] rect{
            fill: ${val} !important;
        }

        g[sectorId="${prop}"] text{
            fill: #fff !important;
        }`;
      })
      .join("\r\n");
    styleElement.value.innerHTML = newStyles;
  }
};

onMounted(() => {
  styleElement.value = document.createElement("style");
  styleElement.value.type = "text/css";
  document.head.appendChild(styleElement.value);
  updateStyles();
});

onUnmounted(() => {
  if (styleElement.value) {
    document.head.removeChild(styleElement.value);
  }
});

watch(() => sectorToColor, updateStyles, { deep: true });
</script>