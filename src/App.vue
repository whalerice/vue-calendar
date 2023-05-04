<template>
  <FullCalendar :locale="currentLocale">
    <template #dateCellRender="{ current }">
      <ul>
        <li>{{ current }}</li>
      </ul>
    </template>
  </FullCalendar>
</template>
<script setup lang="ts">
import FullCalendar from '@/components/FullCalendar.vue';
import { ref } from 'vue';

const currentLocale = ref<string>('ko');
</script>
