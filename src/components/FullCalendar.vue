<template>
  <div class="calendar-area">
    <div class="calendar-header">
      <button type="button" class="btn-date-lt" @click="calendarData(-1)">
        <i class="arrow"></i>
      </button>
      <div>{{ dayjs(new Date(currentYear, currentMonth - 1)).format('YYYY-MM') }}</div>
      <button type="button" class="btn-date-rt" @click="calendarData(1)">
        <i class="arrow"></i>
      </button>
    </div>
    <table class="calendar">
      <thead>
        <tr>
          <td v-for="(week, index) in weeks" :key="index">
            {{ week }}
          </td>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(row, index) in dates" :key="index">
          <td
            v-for="(item, index2) in row"
            :key="index2"
            :class="{
              'text-light': dayjs(item).month() + 1 !== currentMonth,
              today: item === today,
            }"
          >
            <span class="date">
              {{ dayjs(item).date() }}
            </span>
            <div class="panel">
              <slot name="date" :current="item"></slot>
            </div>
          </td>
          <td v-if="props.space" class="week-total">
            <div class="panel">
              <slot name="weekTotal" :current="row"></slot>
            </div>
          </td>
        </tr>
        <tr v-if="props.space">
          <td v-for="num in 7" :key="num">
            <slot :name="`total${num}`"></slot>
          </td>
          <td class="total">
            <slot name="allTotal"></slot>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script setup lang="ts">
import '@/scss/calendar.scss';
import 'dayjs/locale/de';
import 'dayjs/locale/ko';
import 'dayjs/locale/zh-cn';
import 'dayjs/locale/ja';
import localeData from 'dayjs/plugin/localeData';
import dayjs from 'dayjs';
import { onBeforeMount, onMounted, ref } from 'vue';

const props = defineProps({
  locale: {
    type: String,
    default: '',
  },
  weekdays: {
    type: String,
    default: '',
  },
  space: {
    type: String || null,
    default: null,
  },
});

dayjs.locale(props.locale);
dayjs.extend(localeData);

const date = new Date();
const today = ref<string>(dayjs(date).format('YYYY-MM-DD'));
const weeks = ref<string[]>([...dayjs.weekdays()]);
const dates = ref<string[][]>([]);
const currentYear = ref<number>(0);
const currentMonth = ref<number>(0);

const calendarData = (arg: number) => {
  if (arg < 0) currentMonth.value -= 1; // -1이 들어오면 지난 달 달력으로 이동
  if (arg === 1) currentMonth.value += 1; // 1이 들어오면 다음 달 달력으로 이동

  if (currentMonth.value === 0) {
    // 작년 12월
    currentYear.value -= 1;
    currentMonth.value = 12;
  } else if (currentMonth.value > 12) {
    // 내년 1월
    currentYear.value += 1;
    currentMonth.value = 1;
  }
  const [monthFirstDay, monthLastDate, lastMonthLastDate] = getFirstDayLastDate(
    currentYear.value,
    currentMonth.value,
  );
  dates.value = getMonthOfDays(monthFirstDay, monthLastDate, lastMonthLastDate);
};
const getFirstDayLastDate = (year: any, month: any) => {
  const firstDay = new Date(year, month - 1, 1).getDay(); // 이번 달 시작 요일
  const lastDate = new Date(year, month, 0).getDate(); // 이번 달 마지막 날짜
  let lastYear = year;
  let lastMonth = month - 1;
  if (month === 1) {
    lastMonth = 12;
    lastYear -= 1;
  }

  const prevLastDate = new Date(lastYear, lastMonth, 0).getDate(); // 지난 달 마지막 날짜
  return [firstDay, lastDate, prevLastDate];
};
const getMonthOfDays = (
  monthFirstDay: number,
  monthLastDate: number,
  prevMonthLastDate: number,
) => {
  let day = 1;
  let prevDay = prevMonthLastDate - monthFirstDay + 1;

  const dates: any = [];
  let weekOfDays = [];
  while (day <= monthLastDate) {
    if (day === 1) {
      // 1일이 어느 요일인지에 따라 테이블에 그리기 위한 지난 셀의 날짜들을 구할 필요가 있다.
      for (let j = 0; j < monthFirstDay; j += 1) {
        weekOfDays.push(
          dayjs(new Date(currentYear.value, currentMonth.value - 2, prevDay)).format('YYYY-MM-DD'),
        );
        prevDay += 1;
      }
    }

    weekOfDays.push(
      dayjs(new Date(currentYear.value, currentMonth.value - 1, day)).format('YYYY-MM-DD'),
    );

    if (weekOfDays.length === 7) {
      // 일주일 채우면
      dates.push(weekOfDays);
      weekOfDays = []; // 초기화
    }
    day += 1;
  }

  if (weekOfDays.length > 0) dates.push(weekOfDays); // 남은 날짜 추가

  const len = weekOfDays.length;
  if (len > 0 && len < 7) {
    for (let k = 1; k <= 7 - len; k += 1) {
      weekOfDays.push(
        dayjs(new Date(currentYear.value, currentMonth.value, k)).format('YYYY-MM-DD'),
      );
    }
  }
  return dates;
};

onBeforeMount(() => {
  currentYear.value = date.getFullYear();
  currentMonth.value = date.getMonth() + 1;

  if (props.weekdays === 'short') {
    weeks.value = [...dayjs.weekdaysShort()];
  }

  if (props.space) {
    weeks.value = [...dayjs.weekdaysShort(), props.space];
  }
});

onMounted(() => {
  calendarData(0);
});
</script>
