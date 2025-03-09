<script setup>
import Scheme from '@/components/Scheme.vue';
import mockData from '@/mock2';
</script>

<template>
  <header>
    <!-- None -->
  </header>

  <main>
    <Scheme :="{ ...mockData }"/>
  </main>
</template>

<style scoped>
/* none */
</style>
