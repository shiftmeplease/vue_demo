<template>
    <div class="wrapper">
        <div class="schemeContainer" ref="containerRef">
            <gMap v-if="schemeData.length > 0" :schemeData="schemeData" :sectorToColor="cutSectorColors"
                :highlightId="highlightId" @hover="handleChildHover" @leave="handleChildLeave"
                @click="handleChildClick"></gMap>
        </div>
        <tooltip v-if="showTooltip" :x="tooltipX" :y="tooltipY">
            {{ tooltipText }}
        </tooltip>
    </div>  
</template>

<style>
.wrapper {
    position: relative;
}

g[sectorId] {
    filter: drop-shadow(0px 0px 6px #ffffff00);
    transition: filter ease-in 0.2s;
}

.schemeContainer {
    margin: 0;
    font-family: "Nunito", sans-serif;
    font-size: 0.875rem;
    font-weight: 400;
    line-height: 1.5;
    color: #212529;
    background-color: #fff;
    -webkit-text-size-adjust: 100%;
    -webkit-tap-highlight-color: transparent;
    overflow: hidden;
    display: flex;
    justify-content: center;
}

*,
::after,
::before {
    box-sizing: border-box;
}

svg text {
    -webkit-touch-callout: none;
    -webkit-user-select: none;
    -khtml-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
}

.modal-additional {
    top: 33vh;
}

.modal-overlay {
    background-color: #21252977;
    width: 100%;
    height: 100%;
}

.filter-gray {
    filter: grayscale();
}

.filter-active {
    filter: blur(10px);
}

.selected path,
.selected rect {
    stroke: #ff912c;
    stroke-width: 1;
    fill: #e3e7ea !important;
    transition: stroke 10s ease-out, fill 0.5s ease-out;
}

.selected path:focus,
.selected rect:focus {
    fill: #fff;
    stroke: #fff !;
    stroke-width: 1;
}

.circle {
    display: inline-block;
    width: 10px;
    height: 10px;
    border-radius: 5px;
    margin: 0;
}

#modal {
    /* display: block; */
}

/* Sectors */

/* Places */

html {
    scroll-behavior: smooth;
}

/* #return,*/
#paintPlacesAll,
#paintPlacesAllSecond {
    color: #000;
    cursor: pointer;
    z-index: 999;
    font-size: 10px;
    display: flex;
}

/* #return i,*/
#paintPlacesAll i,
#paintPlacesAllSecond i {
    font-size: 1.1em;
    line-height: inherit;
}

/* #return{
      margin-right: 1.5em;
  } */

.additionalEdit {
    display: flex;
    padding: 10px;
    flex-flow: row nowrap;
    gap: 10px;
    margin-bottom: -27.5px;
    background-color: #fff;
    border-radius: 5px 5px 0 0;
    width: fit-content;
    padding-left: 35px;
}

/* @media (width <=1000) {
      #return {
          position: static;
          bottom: 0;
          left: 0;
      }
  } */

.actionZoom {
    position: absolute;
    top: 0;
    right: 0px;
    z-index: 999;
    color: #000;
    z-index: 999;
}

.actionZoom button {
    width: 80px;
    background-color: #fff;
    padding: 10px;
    outline: none;
    border: none;
    border: 3px solid pink;
    cursor: pointer;
    font-size: 24px;
    font-weight: 700;
    transition: background-color 2s;
    user-select: none;
    color: #000;
}

.actionZoom button:hover {
    background-color: #8ab9ef;
}

#minusSize {
    margin-top: 70px;
}

.tooltipX {
    background: #fff;
    border-radius: 5px;
    box-shadow: 0 0 20px 0 rgba(0, 0, 0, 0.25);
    padding: 15px;
    text-align: center;
    color: #8f9da6;
    width: 250px;
}

.tooltipX>span {
    display: block;
    font-size: 18px;
    font-weight: 400;
    color: #7c7c7c;
}

.tooltipPlace {
    position: absolute;
    display: none;
    background: #fff;
    border-radius: 5px;
    box-shadow: 0 0 20px 0 rgba(0, 0, 0, 0.25);
    padding: 15px 44px;
    text-align: center;
    color: #8f9da6;
}

.tooltipPlace>span {
    display: block;
    font-size: 18px;
    font-weight: 400;
    color: #7c7c7c;
}

/* Editor Mode */

.tooltipLineEditor,
.tooltipPlaceEditor {
    position: absolute;
    top: 0;
    right: 0;
    display: none;
    background-color: rgba(255, 255, 255, 1);
    color: #000;
    padding: 10px;
    user-select: none;
}

.tooltipLineEditor h3,
.tooltipPlaceEditor h3 {
    background: #f3f3f3;
    padding: 5px;
    margin: 0;
    text-align: center;
}

.tooltipLineEditor ul,
.tooltipPlaceEditor ul {
    list-style: none;
    padding: 0;
    margin: 0;
}

.tooltipLineEditor ul li,
.tooltipPlaceEditor ul li {
    background-color: #1967a9;
    margin: 2px 0px 2px 0px;
    color: #fff;
    padding: 10px;
    cursor: pointer;
}

.tooltipLineEditor .actions,
.tooltipPlaceEditor .actions {
    background: #f3f3f3;
    color: #000;
    font-size: 24px;
    text-align: center;
    width: 100%;
}

.tooltipLineEditor .actions i,
.tooltipPlaceEditor .actions i {
    cursor: pointer;
    user-select: none;
}

.tooltipPlaceEditor {
    margin-top: 280px;
}

.tooltipCreateLine {
    position: absolute;
    top: 60px;
    left: 0;
    display: none;
    background-color: rgba(255, 255, 255, 1);
    color: #000;
    padding: 10px;
}

.tooltipSaveSector {
    position: absolute;
    top: 0;
    left: 130px;
    display: none;
    background-color: rgba(255, 255, 255, 1);
    color: #000;
    padding: 10px;
}
</style>

<script setup>
import gMap from "@/components/gMapTest.vue";
import Tooltip from "@/components/MapComponents/Tooltip.vue";
import { computed, ref, reactive,onBeforeMount, watch } from "vue";

//   import { numberToLocale } from "../../composables/utils"; >> mock  
const numberToLocale = (v) => { return v }

import mockServerResponse from '@/mock.js' 

const emit = defineEmits(["click"]);

const { groups, schemeId, sectorColors, tData, transactions } = defineProps({
    schemeId: Number,
    groups: Object,
    sectorColors: Object,
    tData: Array,
    transactions: Object,
});

const showTooltip = ref(false);
const tooltipX = ref(0);
const tooltipY = ref(0);
const tooltipText = ref("");
const containerRef = ref(null);

const hoverOffEffectStyle = (clr = "#ffffff00") =>
    `filter: drop-shadow(0px 0px 10px ${clr});`;
const hoverOnEffectStyle = (clr = "#ff00cc") =>
    `filter: drop-shadow(0px 0px 10px ${clr}) hue-rotate(5deg);`;

const hoverSectorId = ref("");
const highlightId = ref("");

const boughtBySector = computed(() => {
    const sectors = {};
    tData.forEach(({ coreId: sectorId, id }) => {
        if (sectorId === -1) {
            //TODO Parking
            return;
        }
        const place = { sectorId, id: id || -1 };
        if (!sectors[sectorId]) {
            sectors[sectorId] = [];
        }
        sectors[sectorId].push(place);
    });
    return sectors;
});

const cutSectorColors = computed(() => {
    const result = {};
    function calculatePercentage(smallest, largest, number) {
        if (largest === smallest) {
            throw new Error("Largest and smallest values cannot be the same.");
        }

        // Calculate the percentage
        const percentage = ((number - smallest) / (largest - smallest)) * 100;

        // Reverse the percentage
        const reversedPercentage = 100 - percentage;

        return reversedPercentage === 0 ? 0 : reversedPercentage;
    }

    function calculateLogarithmicPercentage(smallest, largest, number) {
        if (largest === smallest) {
            throw new Error("Largest and smallest values cannot be the same.");
        }

        // Normalize the number to a value between 0 and 1
        const normalizedValue = (number - smallest) / (largest - smallest);

        // Apply a logarithmic scale
        const logScale = Math.log10(normalizedValue + 1) / Math.log10(2); // Using log base 10 for scaling

        // Convert to percentage and reverse it
        const reversedPercentage = (1 - logScale) * 100;

        return reversedPercentage;
    }

    function calculateExponentialPercentageWithBase(
        smallest,
        largest,
        number,
        base = 2
    ) {
        if (largest === smallest) {
            throw new Error("Largest and smallest values cannot be the same.");
        }
        if (number < smallest || number > largest) {
            throw new Error("Number must be between smallest and largest values.");
        }

        // Normalize the number to a value between 0 and 1
        const normalizedValue = (number - smallest) / (largest - smallest);

        // Apply an exponential scale with the given base
        const expScale = (Math.pow(base, normalizedValue) - 1) / (base - 1);

        // Convert to percentage and reverse it
        const reversedPercentage = (1 - expScale) * 100;

        return reversedPercentage;
    }

    function normalizeValue(smallest, largest, number) {
        if (largest === smallest) {
            throw new Error("Largest and smallest values cannot be the same.");
        }
        return (number - smallest) / (largest - smallest);
    }

    function applyExponentialScale(normalizedValue, base = 2) {
        return (Math.pow(base, normalizedValue) - 1) / (base - 1);
    }

    function interpolateColor(startColor, endColor, percent) {
        const r = Math.round(startColor.r + percent * (endColor.r - startColor.r));
        const g = Math.round(startColor.g + percent * (endColor.g - startColor.g));
        const b = Math.round(startColor.b + percent * (endColor.b - startColor.b));

        return { r, g, b };
    }

    function rgbToHex({ r, g, b }) {
        const toHex = (n) => n.toString(16).padStart(2, "0").toUpperCase();
        return `#${toHex(r)}${toHex(g)}${toHex(b)}`;
    }

    // function getHeatmapColor(smallest, largest, number, base = 2) {
    //   const normalizedValue = normalizeValue(smallest, largest, number);
    //   const scaledValue = applyExponentialScale(normalizedValue, base);

    //   const startColor = { r: 90, g: 175, b: 64 }; // green
    //   const endColor = { r: 196, g: 16, b: 16 }; // Red

    //   const color = interpolateColor(startColor, endColor, scaledValue);
    //   return rgbToHex(color);
    // }

    function getHeatmapColor(smallest, largest, number, base = 2) {
        const normalizedValue = normalizeValue(smallest, largest, number);
        const scaledValue = applyExponentialScale(normalizedValue, base);

        const grey = { r: 193, g: 193, b: 193 }; // Grey
        const yellow = { r: 226, g: 213, b: 31 }; // Yellow
        const red = { r: 196, g: 16, b: 16 }; // Red

        let color;

        if (scaledValue <= 0.5) {
            // Interpolate between grey and yellow
            color = interpolateColor(grey, yellow, scaledValue * 2);
        } else {
            // Interpolate between yellow and red
            color = interpolateColor(yellow, red, (scaledValue - 0.5) * 2);
        }

        return rgbToHex(color);
    }

    const values = Object.values(boughtBySector.value).map((v) => v.length);
    const largest = Math.max(...values); //from max
    // const largest = values.reduce((accumulator, currentValue) => {
    //   return accumulator + currentValue;
    // }, 0); //from Summ of all tickets
    const smallest = 0;
    for (let sectorId in boughtBySector.value) {
        // const largest = placesBySectorId.value[sectorId].length; //TODO is it better, to heatmap by places in sector or overall?
        let bought = boughtBySector.value[sectorId].length;

        result[sectorId] = getHeatmapColor(smallest, largest, bought, 0.01); //1.5
    }
    for (let sectorId of activeSectorIds.value) {
        if (!result[sectorId])
            result[sectorId] = getHeatmapColor(smallest, largest, 0, 2);
    }
    return result;
});

const handleChildHover = (sectorId, event) => {

    const wrapperElem = containerRef.value;
    const rect = wrapperElem.getBoundingClientRect();

    if (!activeSectorIds.value.includes(sectorId)) return;
    const sectorPlaces = placesBySectorId.value[sectorId];
    if (sectorPlaces === void 0) return;

    showTooltip.value = true;

    tooltipX.value = event.clientX - rect.left;
    tooltipY.value = event.clientY - rect.top;

    const forSaleAmt = sectorPlaces.length;
    const minPrice = minPricesBySectorId.value[sectorId];

    const saleType = sectorPlaces.some((v) => v.placeId == -1)
        ? "Сектор"
        : "Места";
    const boughtInSector = boughtBySector.value[sectorId];
    const boughtTotalInSector = transactions.reduce((total, tx) => {
        const { tickets } = tx.details;
        tickets.forEach(({ coreId, price }) => {
            if (coreId != sectorId) return;
            total += price
        });

        return total;
    }, 0);

    const boughtAmt = boughtInSector ? boughtInSector.length : 0;
    tooltipText.value = `Куплено: ${numberToLocale(boughtAmt)} | Выручка: ${numberToLocale(boughtTotalInSector)} | К продаже: ${numberToLocale(forSaleAmt)} | Тип продажи: ${saleType}`;
    // tooltipText.value = `Element ID: ${sectorId}, Цена:${minPrice}, К продаже: ${forSaleAmt}, Тип продажи: ${saleType}, Куплено: ${boughtAmt}`;
    document.querySelector(`g[sectorId="${sectorId}"]`).style =
        hoverOnEffectStyle(cutSectorColors.value[sectorId]);
};

let prevClickId = null;
const handleChildClick = (sectorId) => {
    alert(`sectorId cliked: ${sectorId}; Mocked version, doesnt support this action`);
    return 

    if (!activeSectorIds.value.includes(sectorId)) return;
    const sectorPlaces = placesBySectorId.value[sectorId];
    if (sectorPlaces === void 0) return;

    const isSector = sectorPlaces.some((v) => v.placeId == -1);

    if (prevClickId) {
        document.querySelector(`g[sectorId="${prevClickId}"]`).style =
            hoverOffEffectStyle;
    }
    if (prevClickId === sectorId) {
        emit("click", null, null);
    } else {
        document.querySelector(`g[sectorId="${sectorId}"]`).style =
            hoverOnEffectStyle(cutSectorColors.value[sectorId]);
        emit("click", sectorId, isSector);
    }
    prevClickId = prevClickId === sectorId ? null : sectorId;
};

const handleChildLeave = (sectorId) => {

    if (prevClickId != sectorId)
        document.querySelector(`g[sectorId="${sectorId}"]`).style =
            hoverOffEffectStyle;

    hoverSectorId.value = "";
    showTooltip.value = false;
};

const schemeData = reactive([]);
const activeSectorIds = ref([]);
const placesBySectorId = ref({});
const minPricesBySectorId = ref({});

// const cutSectorColors = computed(() => {
//   if (!sectorColors) return {};

//   return Object.entries(sectorColors).reduce((accum, [sectorId, info]) => {
//     const { price, color } = info;
//     accum[sectorId] = color;
//     return accum;
//   },{});
// });

onBeforeMount(async () => {

    const { scheme } = mockServerResponse;
    schemeData.length = 0;
    schemeData.push(...scheme);
});

function handleActiveClass(schemeElem, sectorId, activeArr) {
    if (schemeElem.active == 1) {
        activeArr.push(sectorId);
    }
}

function handleGroupData(sectorId, placesByCoreId) {
    const sectorPlacesWithGroups = [];

    groups.list.forEach((group) => {
        // if (group.editType != 1) return;

        let sectorPlaces = group.places.filter(
            (place) => place.coreId === sectorId
        );

        if (sectorPlaces.length > 0) {
            const amtOfUnorderedPlaces = sectorPlaces.filter(
                (v) => v.placeId == -1
            ).length;
            const haveUnordered = amtOfUnorderedPlaces > 0;
            const haveOrderedPlaces = sectorPlaces.some((v) => v.placeId !== -1);
            const mixedPlaces = haveUnordered && haveOrderedPlaces;

            sectorPlaces.forEach((place) => {
                place.group = {
                    sectorId: place.coreId,
                    price: group.price,
                    color: group.color,
                };
            });

            if (mixedPlaces)
                sectorPlaces = sectorPlaces.filter((v) => v.placeId == -1);

            sectorPlacesWithGroups.push(...sectorPlaces);
        }
    });

    if (sectorPlacesWithGroups.length >= 1) {
        placesByCoreId[sectorId] = sectorPlacesWithGroups;
    }
}

watch(
    schemeData,
    (schemeData) => {
        let sectorSet = new Set();
        const activeArr = [];
        // const sectorData = {};
        const groupPlacesBySectorId = {};
        schemeData.forEach((schemeElem) => {
            const sectorId = +schemeElem.core;

            if (sectorSet.has(sectorId)) {
                return;
            }
            sectorSet.add(sectorId);

            handleActiveClass(schemeElem, sectorId, activeArr);

            if (!activeArr.includes(sectorId)) return;
            handleGroupData(sectorId, groupPlacesBySectorId);
            // handleSectorData(sectorId, sectorData);
            // handlePlacesByCoreId(sectorId);
            // handleSectorSeats(sectorId);
        });

        placesBySectorId.value = groupPlacesBySectorId;
        minPricesBySectorId.value = Object.keys(groupPlacesBySectorId).reduce(
            (obj, sectorId) => {
                const sectorPrice = groupPlacesBySectorId[sectorId].reduce(
                    (minPrice, place) => {
                        const placePrice = place.group.price;
                        return minPrice > placePrice ? placePrice : minPrice;
                    },
                    Infinity
                );
                obj[sectorId] = sectorPrice;
                return obj;
            },
            {}
        );
        activeSectorIds.value = activeArr;
    },
    { deep: true }
);
</script>