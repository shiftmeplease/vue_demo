<template>
  <div>
    <h1 class="sectorName" v-if="sectorName">Сектор {{ sectorName }}</h1>
    <div class="mapPlaces" ref="containerOfElem" v-if="tree">
      <!-- workaround so it is same time as drawing -->
      <div id="testZoom" class="containerX" ref="zoomableElem">
        <div v-for="line in tree" :key="line.id" class="line">
          <div :class="line.name === 'empty' ? 'emptyTitle' : 'title'">
            {{ (line.name !== "empty" && line.name) || "&nbsp;" }}
          </div>
          <div
            v-if="line.offset && line.offset > 0"
            :style="`min-width:${line.offset * seatWidth}px`"
          ></div>

          <template v-for="place in line.places">
            <div
              v-if="place.offset && place.offset > 0"
              :style="`min-width:${place.offset * seatWidth}px`"
              :key="place.id + 'offset'"
            ></div>

            <template v-if="place.group">
              <div
                class="place"
                :data-id="place.id"
                :style="`background: ${groupedColors[place.id] || '#d3d3d3'};`"
                :class="place.bought && 'bought-place'"
                :key="place.id"
                @mousemove.self="(ev) => handleMouseEnter(place.id, ev)"
                @mouseleave.stop="(ev) => handleMouseLeave(place.id, ev)"
              >
                {{ place.name }}
              </div>
            </template>
            <template v-else>
              <div
                class="placeBusy"
                :data-id="place.id"
                style="background: #d3d3d3; color: #fff"
                :key="place.id"
              >
                &nbsp;
              </div>
            </template>
          </template>

          <div :class="line.name === 'empty' ? 'emptyTitle' : 'title'">
            {{ (line.name !== "empty" && line.name) || "&nbsp;" }}
          </div>
        </div>
      </div>
      <tooltip v-if="showTooltip" :x="tooltipX" :y="tooltipY" ref="tooltipRef">
        {{ tooltipText }}
      </tooltip>
    </div>
  </div>
</template>

<style scoped>
.mapPlaces {
  --mapSize: 600px;
}

.line {
  /* --placesmargin: 10px;
   --placespadding: 3px;
   --lineMargin: 10px 0px;
    --fontSize: 14px */
  --placesmargin: 0;
  --placespadding: 3px;
  --lineMargin: 5px 0px;
  --fontSize: 18px;
  --lineHeight: 42px;
  --seatSize: 35px;
}

.mapPlaces {
  position: relative;
  display: flex;
  flex-direction: column;
  width: 100%;
  max-height: var(--mapSize);
  height: 100%;
  display: flex;
  background-color: #fff;
  /* justify-content: flex-start; */
  /* align-items: center; */
  overflow: hidden;
}

.sectorName {
  padding-top: 10px;
  text-align: center;
  background-color: #fff;
  margin: 0;
}

.containerX {
  /* display: block; */
  display: flex;
  flex-direction: column;
  align-items: center;
  position: relative;
  /* margin: 0 auto; */
  padding: 3vw;
  user-select: none;
  -webkit-user-select: none;
  -moz-user-select: none;
  user-select: none;
  height: 100%;
  /* width: 100%; */
}

.line {
  display: flex;
  flex-direction: row;
  align-items: center;
  margin: var(--lineMargin);
  position: relative;
  justify-content: flex-start;
  cursor: pointer;
}

.line .title {
  font-size: 32px;
  font-weight: bold;
  /* width: 100px; */
  min-width: 100px;
  color: #939393;
  line-height: var(--lineHeight);
}

.line .title:first-child {
  text-align: left;
  margin-right: 10px;
}

.line .title:last-child {
  text-align: right;
  margin-left: 10px;
}

.line .emptyTitle {
  font-size: 32px;
  font-weight: bold;
  min-width: 100px;
  color: #939393;
  height: var(--lineHeight);
}

.place {
  background-color: #ededed;
  border: 1px solid #bbb;
  color: #000;
  padding: var(--placespadding);
  margin-left: var(--placesmargin);
  margin-right: var(--placesmargin);
  font-size: var(--fontSize);
  min-width: var(--seatSize);
  font-weight: bold;
  text-align: center;
  -webkit-user-select: none;
  -moz-user-select: none;
  user-select: none;
  cursor: pointer;
  position: relative;
}

.line .placeEmpty {
  /*background-color: black; */
  color: #fff;
  /* padding: 5px; */
  margin-left: var(--placesmargin);
  margin-right: var(--placesmargin);
  height: var(--fontSize);
  min-width: var(--seatSize);
  font-weight: bold;
}

.line .placeBusy {
  background-color: #ededed;
  border: 1px solid #bbb;
  color: #000;
  padding: var(--placespadding);
  margin-left: var(--placesmargin);
  margin-right: var(--placesmargin);
  font-size: var(--fontSize);
  font-weight: bold;
  text-align: center;
  -webkit-user-select: none;
  -moz-user-select: none;
  user-select: none;
  cursor: pointer;
  min-width: var(--seatSize);
}
/*
.bought-place::after {
  content: "X";  
  color: rgb(255, 255, 255);
  font-size: 1.5em;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  opacity: 55%;
}*/
.bought-place {
  /* filter: drop-shadow(0px 0px 2px #3f0); */
  color: white;
}
</style>

<script setup>
import Tooltip from "@/components/MapComponents/Tooltip.vue";
import {
  ref,
  onMounted,
  onBeforeUnmount,
  nextTick,
  toRefs,
} from "vue";
import panzoom from "panzoom"; // Make sure to install panzoom

import { numberToLocale } from "../../composables/utils.ts";
import { stringToColor } from "../../composables/utils";
import { useRuntimeConfig } from '#imports';

const config = useRuntimeConfig();

const zoomableElem = ref(null);
const containerOfElem = ref(null);
let panZoomer = null;
const tooltipRef = ref(null);

const props = defineProps({
  schemeId: { type: Number, required: true },
  sectorId: { type: Number, required: true },
  tData: { type: Array },
  placesInfo: { type: Object },
  transactions: { type: Array },
  sectorName: {type: String}
});
const { schemeId, tData, placesInfo, transactions } = props;

const { sectorId, sectorName } = toRefs(props);

const groupedColors = reactive({})

const initializePanZoom = () => {
  const scaleX =
    containerOfElem.value.offsetWidth / zoomableElem.value.offsetWidth;
  const scaleY =
    containerOfElem.value.offsetHeight / zoomableElem.value.offsetHeight;
  const initialZoom = Math.min(scaleX, scaleY);

  panZoomer = panzoom(zoomableElem.value, {
    maxZoom: 2.0,
    minZoom: initialZoom,
    initialZoom,
    bounds: true,
    boundsPadding: 0.2,
    boundsDisabledForZoom: false,
    zoomDoubleClickSpeed: 0,
    contain: "inside",
    transformOrigin: { x: 0.5, y: 0.5 },
    beforeWheel: (e) => !e.altKey,
    onDoubleClick: (e) => {
      e.preventDefault();
      throw new Error("no zoom is allowed!");
    },
    beforeMouseDown: (e) => !e.altKey,
  });

  const computedWidth = zoomableElem.value.offsetWidth * initialZoom;
  const computedHeight = zoomableElem.value.offsetHeight * initialZoom;
  const centerX = (containerOfElem.value.offsetWidth - computedWidth) / 2;
  const centerY = (containerOfElem.value.offsetHeight - computedHeight) / 2;
  panZoomer.moveTo(centerX, centerY);
};

const handleResize = () => {
  if (panZoomer) {
    panZoomer.dispose();
  }
  initializePanZoom();
};

const fetchSectorData = async (sector) => {
  alert('Mocked version, doesnt support this action');

  const linesData = response.data.lines ?? null;
  const placesData = response.data.all ?? null;
  if (!linesData) return;

  linesData.forEach((element) => {
    if (element.var !== "empty") element.var = parseInt(element.var);
  });
  linesData.sort((a, b) => b.position - a.position);

  lines.value = linesData;
  places.value = placesData;
};

// onBeforeMount(() => {
//   fetchSectorData(sectorId.value);
// });

const highlightGroups = () => {
  transactions.forEach((tx) => {
    const { tickets } = tx.details;

    tickets.forEach(({ id }) => {
      if (!id) return;
      const seatElem = document.querySelector(
        `.place.bought-place[data-id="${id}"]`
      );
      if (seatElem)
      groupedColors[id] = stringToColor(
          tx.details.email + tx.details.phone
        )
        // seatElem.style.outline = `5px solid ${stringToColor(
        //   tx.details.email + tx.details.phone
        // )}`; //TODO id
    });
  });
};

watch(
  sectorId,
  (newsectorId) => {
    if (!newsectorId) return;
    fetchSectorData(newsectorId);
  },
  { immediate: true }
);

onBeforeUnmount(() => {
  if (panZoomer) {
    panZoomer.dispose();
  }
  window.removeEventListener("resize", handleResize);
});

onMounted(() => {
  initializePanZoom();
  window.addEventListener("resize", handleResize);
});

const showTooltip = ref(false);
const tooltipX = ref(0);
const tooltipY = ref(0);
const tooltipText = ref("");

const handleMouseEnter = (placeId, event) => {
  //   if (!activeSectorIds.value.includes(sectorId)) return;
  //   const sectorPlaces = placesBySectorId.value[sectorId];
  //   if (sectorPlaces === void 0) return;
  const { price } = placesInfo[sectorId.value][placeId];
  // let mockSwitch = false;
  const transaction = transactions.find((tx) => {
    const { tickets } = tx.details;
    const isFound = tickets.find(({ id }) => id === placeId);
    return isFound
  });
  let addInfo = "";

  if (transaction) {
    const { name, phone } = transaction;
    // addInfo = `Имя: ${name} | +7${phone} | Цена Покупки: ${transaction.sums[itemIndex]}`;
    // addInfo = `| Покупка: ${numberToLocale(transaction.sums[itemIndex])}`;
    const boughtInSector = transaction.details.tickets.reduce(
      (accum, ticket) => {
        const placeId = ticket.id;
        const inThisSector = places.value.find((value) => {
          return value.spId == placeId;
        });
        if (inThisSector) {
          accum.amount++;
          accum.prices.push(ticket.price);
        }
        return accum;
      },
      { amount: 0, prices: [] }
    );

    let total = 0;

    const stringPrices = boughtInSector.prices
      .map((v) => {
        total += v;
        return numberToLocale(v);
      })
      .join(" + ");

    addInfo = `Билетов: ${boughtInSector.amount} | Покупка: ${stringPrices}${
      stringPrices.includes("+") ? " = " + numberToLocale(total) : ''
    }`; //TODO filter?
  }

  // tooltipText.value = `placeId: ${placeId} | Цена: ${price} ${
  //   (transaction && addInfo) || ""
  // }`;
  // tooltipText.value = `Цена: ${numberToLocale(price)} ${
  //   (transaction && addInfo) || ""
  // }`;
  tooltipText.value = (transaction && addInfo) || "Нет информации о покупке";

  showTooltip.value = true;

  const wrapperElem = containerOfElem.value;
  const rect = wrapperElem.getBoundingClientRect();

  tooltipX.value = event.clientX - rect.left;
  tooltipY.value = event.clientY - rect.top;
};

const handleMouseLeave = (placeId, ev) => {
  showTooltip.value = false;
};

const seatWidth = 35;
const lines = ref([]);
const places = ref([]);

const tree = ref([]);

function computeTree(lines) {
  const tree = [];

  for (let line of lines) {
    let linePlaces = places.value.filter((value) => {
      return value.schemeIdx == line.id;
    });
    linePlaces = [
      ...new Map(linePlaces.map((item) => [item["id"], item])).values(),
    ];
    linePlaces.sort((a, b) => a.sPosition - b.sPosition);
    const { var: lineName, position, id, offset } = line;
    const treeEntry = { name: lineName, position, id, offset };
    treeEntry.places = linePlaces.map((place) => {
      let placeData = places.value.find((v) => v.id == place.spId);
      let placeGroupInfo;
      try {
        placeGroupInfo = placesInfo[sectorId.value][place.spId];
      } catch (err) {
        console.log(err, "no group");
      }
      let placeBought = tData.some((v) => v.id == place.spId);

      if (placeData) {
        const placeEntry = {
          id: placeData.spId,
          name: placeData.sVar,
          line: lineName,
          position: placeData.sPosition,
          offset: placeData.sOffset,
          bought: placeBought,
        };

        if (placeGroupInfo) {
          placeEntry.group = placeGroupInfo;
          //price color
        }
        return placeEntry;
      }
    });
    tree.push(treeEntry);
  }
  return tree;
}

watch(lines, (lines) => {
  tree.value = computeTree(lines);
  nextTick(() => {
    handleResize();
    highlightGroups();
  });
});

// watch(
//   tree,
//   (newTree, oldTree) => {
//     console.log("Tree data updated:", newTree);
//     // Perform any actions needed when the tree data is updated
//     if (panZoomer) {
//       panZoomer.dispose();
//     }
//     initializePanZoom();
//   },
//   { deep: true }
// );
</script>