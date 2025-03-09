<!-- <template>
  <div class="parkingContainer p-3 mt-2">
    <template v-if="parking && parking.length > 0">
      Парковки:
      <div
        class="d-flex gap-2 justify-content-evenly align-items-center flex-wrap flex-column"
      >
        <div v-for="park in parking" :key="park.id" class="park">
          <span style="font-weight: bold">{{ park.name }} </span
          ><span v-if="parkingSaleInfo[park.id]">
            Куплено: {{ parkingSaleInfo[park.id].count }} Сумма:
            {{ parkingSaleInfo[park.id].total }}
          </span>
        </div>
      </div>
    </template>

    <div v-else>Парковки отсутствуют</div>
  </div>
</template> -->

<template>
  <div class="parkingContainer p-3 mt-2">
    <template v-if="parking && parking.length > 0">
      <span class="bold">Парковки</span>
      <table class="parkingTable">
        <thead>
          <tr>
            <th>Название</th>
            <th>Куплено</th>
            <th>Сумма</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="park in parking" :key="park.id" class="park">
            <td><strong>{{ park.name }}</strong></td>
            <td v-if="parkingSaleInfo[park.id]">
              {{ numberToLocale(parkingSaleInfo[park.id].count) }}
            </td>
            <td v-if="parkingSaleInfo[park.id]">
              {{ numberToLocale(parkingSaleInfo[park.id].total) }}
            </td>
          </tr>
        </tbody>
      </table>
    </template>

    <div class="bold" v-else>Парковки отсутствуют</div>
  </div>
</template>

<style scoped>
/* .parkingContainer {
  gap: 20px;
  display: flex;
  flex-direction: row;
  align-items: center;
}

.park {
  display: flex;
  flex-direction: row;
  padding: 10px 16px;
  gap: 8px;
  border-radius: 8px;
  box-shadow: 0px 2px 4px rgba(63, 196, 254, 0.2);
  background: #fff;
  width: 400px;
  justify-content: space-between;
} */

.bold {
  font-weight: bold;
  font-size: 24px;
}

.parkingContainer {
  gap: 20px;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.parkingTable {
  width: 100%;
  border-collapse: collapse;
  /* margin: 20px 0; */
}

.park {
  background: #fff;
  box-shadow: 0px 2px 4px rgba(63, 196, 254, 0.2);
  border-radius: 8px;
}

.park td {
  padding: 10px 16px;
  text-align: left;
  border: 1px solid #ddd;
}
</style>

<script setup>
import { numberToLocale } from '../../composables/utils';

const { parking, transactions } = defineProps({
  parking: Array,
  transactions: Array,
});

const parkingSaleInfo = computed(() => {
  const result = {};
  console.log(transactions);
  transactions.forEach((tx) => {
    const { sectorNames, sums } = tx;
    sectorNames.forEach((parkName, index) => {
      const parkingObj = parking.find(({ name }) => name === parkName);
      if (!result[parkingObj.id])
        result[parkingObj.id] = { count: 0, total: 0 };
      result[parkingObj.id].count++;
      result[parkingObj.id].total += sums[index];
    });
  });
  return result;
});
</script>