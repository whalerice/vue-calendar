<template>
  <FullCalendar :locale="currentLocale">
    <template #date="{ current }">
      <ul>
        <li v-for="(item, index) in list[current]" :key="index">{{ item.content }}</li>
      </ul>
    </template>
  </FullCalendar>

  <FullCalendar :locale="currentLocale" space="정산">
    <template #date="{ current }">
      <ul>
        <li>cash : {{ list2[current]?.cash }}</li>
      </ul>
    </template>
    <template #weekTotal="{ week }">
      <span>{{ week }}주</span> <br />
      {{ weekTotal[week]?.total }}
      <!-- <span>{{ current[0] }}~{{ current[6] }}</span> -->
    </template>
    <template #total1> total1 </template>
    <template #allTotal> All Total </template>
  </FullCalendar>
</template>
<script setup lang="ts">
import FullCalendar from '@/components/FullCalendar.vue';
import { ref } from 'vue';

const currentLocale = ref<string>('ko');

interface IContent {
  color: string;
  content: string;
}

interface IData {
  [key: string]: IContent[];
}

interface IData2 {
  [key: string]: { total: number };
}
interface IData3 {
  [key: string]: { cash: number };
}

const list = ref<IData>({
  ['2023-05-01']: [{ color: 'yellow', content: '내용' }],
  ['2023-05-02']: [
    { color: 'yellow', content: 'test1' },
    { color: 'danger', content: 'test2' },
    { color: 'danger', content: 'test3' },
    { color: 'danger', content: 'test4' },
  ],
  ['2023-05-15']: [{ color: 'yellow', content: '내용' }],
});

const list2 = ref<IData3>({
  ['2023-05-01']: { cash: 10 },
  ['2023-05-02']: { cash: 30 },
  ['2023-05-15']: { cash: 30 },
});
const weekTotal = ref<IData2>({
  1: { total: 1000 },
});
</script>
