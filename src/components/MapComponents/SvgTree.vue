<template>
  <svg
    id="gMap"

    :viewBox="`0 0 ${width}.00 ${height}.00`"
    xmlns="http://www.w3.org/2000/svg"
    ref="svgElem"
  >
    <g>
      <svg-element
        v-for="rootElement in tree"
        :key="rootElement.id || rootElement.subId || 'root'"
        :element="rootElement"
        @hover="handleHover"
        @leave="handleLeave"
        @click="handleClick"
      ></svg-element>
    </g>
  </svg>
</template>

<script setup>
import { ref, onMounted, defineProps, defineEmits,reactive, onUnmounted  } from "vue";
import SvgElement from "./SvgElement.vue";

// const viewBox = `0 0 ${schemeData[0].sWidth}.00 ${schemeData[0].sHeight}.00`;
// const styleBox = `height: ${schemeData[0].sHeight}px; width: ${schemeData[0].sWidth}px; padding-top:50px; padding-bottom:50px;`;
//TODO res
function resizeSVGByHeight(desiredHeight) {
  var svgNode = svgElem.value;
  if (svgNode) {
    // Extract the viewBox values
    const aspectRatio = props.width.value / props.height.value;

    // Calculate the new width to maintain the aspect ratio
    const newWidth = desiredHeight * aspectRatio;

    // Set the new height and width on the SVG element
    svgNode.style.height = `${desiredHeight}px`;
    svgNode.style.width = `${newWidth}px`;
  }
}

// resizeSVGByHeight("gMap", 600);

const props = defineProps({
  schemeData: Object,
  width: Number,
  height: Number,
  styles: String,
});

const emit = defineEmits(["hover", "leave", "click"]);

const handleHover = (elemId, event) => {
  emit("hover", elemId, event);
};

const handleLeave = (elemId) => {
  emit("leave", elemId);
};

const handleClick = (elemId) => {
  emit("click", elemId);
};
const svgElem = ref(null);
const styleElement = ref(null);

const tree = reactive({ children: [] });

function buildElementTree(schemeData) {
  const tree = { children: [] };

  schemeData.forEach((rawElement) => {
    const { core, coreSub, id } = rawElement;

    //removing props by destructure
    const {
      sOld,
      createdAt,
      sOldImg,
      sWidth,
      sHeight,
      schemeId,
      sStyles,
      path,
      text,
      rect,
      circle,
      line,
      polyline,
      positionSubCore,
      name,
      updatedAt,
      coreSubIdx,
      ...element
    } = rawElement;

    // Find or create the core group
    let coreGroup = tree.children.find((group) => group.id === core);

    if (!coreGroup) {
      coreGroup = { id: core, children: [] }; // Use an array for core group children
      tree.children.push(coreGroup);
    }

    const typeObj = {};
    ["path", "text", "rect", "circle", "line", "polyline"].forEach((type) => {
      if (rawElement[type] !== "") {
        typeObj.attributes = JSON.parse(rawElement[type]);
        typeObj.type = type;
        if (name) {
          typeObj.value = name;
        }
      }
    });

    const result = { ...element, ...typeObj, id: id }; // Include the id in the result
    if (coreSub !== -1) {
      // Find or create the subgroup
      let subGroup = coreGroup.children.find(
        (group) => group.subId === coreSub
      );
      if (!subGroup) {
        subGroup = { subId: coreSub, children: [] }; // Use an array for subgroup children
        coreGroup.children.push(subGroup);
      }
      // Add the element to the subgroup's children
      if (positionSubCore && positionSubCore !== -1) {
        subGroup.position = positionSubCore;
      }
      subGroup.children.push(result);
    } else {
      // If no subgroup, add directly to the core group's children
      coreGroup.children.push(result);
    }
  });

  tree.children.forEach((coreGroup) => {
    // coreGroup.children.sort((a, b) => {
    //     // Assuming the position property is directly on the element
    //     // Adjust if the structure is different
    //     return (a.position || 0) - (b.position || 0);
    // });

    coreGroup.children.forEach((subGroup) => {
      if (subGroup.children && Array.isArray(subGroup.children)) {
        subGroup.children.sort((a, b) => {
          return (a.position || 0) - (b.position || 0);
        });
      }
    });
  });

  return tree;
}

onMounted(() => {
  tree.value = buildElementTree(props.schemeData);

  styleElement.value = document.createElement("style");
  styleElement.value.type = "text/css";
  styleElement.value.innerHTML = props.styles;
  // Append the style element to the head
  document.head.appendChild(styleElement.value);
  resizeSVGByHeight(600);
});

onUnmounted(() => {
  // Remove the style element when the component is destroyed
  if (styleElement.value) {
    document.head.removeChild(styleElement.value);
  }
});
</script>
