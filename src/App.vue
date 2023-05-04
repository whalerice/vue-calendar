<template>
  <FullCalendar>
    <template #dateCellRender="{ current }">
      <ul>
        <li>{{ current }}</li>
      </ul>
    </template>
  </FullCalendar>
</template>
<script setup lang="ts">
import FullCalendar from '@/components/FullCalendar.vue';
</script>
