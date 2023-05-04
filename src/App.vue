<template>
  <FullCalendar :locale="currentLocale">
    <template #date="{ current }">
      <ul>
        <li>{{ current }}</li>
      </ul>
    </template>
  </FullCalendar>

  <!-- <FullCalendar :locale="currentLocale" space="정산">
    <template #date="{ current }">
      <ul>
        <li>{{ current }}</li>
      </ul>
    </template>
    <template #weekTotal="{ current }">
      <span>{{ current[0] }}~{{ current[6] }}</span>
    </template>
    <template #total1> total1 </template>
    <template #allTotal> All Total </template>
  </FullCalendar> -->
</template>
<script setup lang="ts">
import FullCalendar from '@/components/FullCalendar.vue';
import { ref } from 'vue';

const currentLocale = ref<string>('ko');
</script>
