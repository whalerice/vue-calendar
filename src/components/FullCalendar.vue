<template>
  <div class="calendar-area">
    <div class="calendar-header">
      <button type="button" @click="calendarData(-1)">{{ '<' }}</button>
      <div>{{ year }}년 {{ month }}월</div>
      <button type="button" @click="calendarData(1)">{{ '>' }}</button>
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
            :class="{ 'text-light': item.month !== month }"
          >
            <span
              class="date"
              :class="{
                today: item.fullDay && today === item.fullDay,
              }"
            >
              {{ item.day }}
            </span>
            <div class="calendar-panel">
              {{ today }} <br />
              <ul>
                <li>{{ item.fullDay }}</li>
                <li>year: {{ item.year }}</li>
                <li>month: {{ item.month }}</li>
                <li>day: {{ item.day }}</li>
              </ul>
            </div>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script setup lang="ts">
import '@/scss/calendar.scss';
import dayjs from 'dayjs';
// import calendar from 'dayjs/plugin/calendar';
import { onBeforeMount, onMounted, ref } from 'vue';
// dayjs.extend(calendar);

interface IDate {
  day: number | null;
  year?: number;
  month?: number;
  fullDay?: string | null;
}

const date = new Date();
const today = ref<any>(dayjs(date).format('YYYY-MM-DD'));
const weeks = ref<string[]>([
  '일요일',
  '월요일',
  '화요일',
  '수요일',
  '목요일',
  '금요일',
  '토요일',
  // '주간',
]);
const dates = ref<IDate[][]>([]);
const currentYear = ref<number>(0);
const currentMonth = ref<number>(0);
const year = ref<number>(0);
const month = ref<number>(0);

const calendarData = (arg?: any) => {
  if (arg < 0) {
    // -1이 들어오면 지난 달 달력으로 이동
    month.value -= 1;
  } else if (arg === 1) {
    // 1이 들어오면 다음 달 달력으로 이동
    month.value += 1;
  }
  if (month.value === 0) {
    // 작년 12월
    year.value -= 1;
    month.value = 12;
  } else if (month.value > 12) {
    // 내년 1월
    year.value += 1;
    month.value = 1;
  }
  const [monthFirstDay, monthLastDate, lastMonthLastDate] = getFirstDayLastDate(
    year.value,
    month.value,
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
        const item = {
          day: prevDay,
          year: year.value,
          month: month.value - 1,
        };
        weekOfDays.push(item);
        prevDay += 1;
      }
    }
    const d = new Date(year.value, month.value - 1, day);
    weekOfDays.push({
      day,
      year: year.value,
      month: month.value,
      fullDay: dayjs(d).format('YYYY-MM-DD'),
    });

    if (weekOfDays.length === 7) {
      // 일주일 채우면
      // if (weeks.value.length > 7) weekOfDays.push(-1);
      dates.push(weekOfDays);
      weekOfDays = []; // 초기화
    }
    day += 1;
  }

  // if (weeks.value.length > 7) weekOfDays.push(-2);
  if (weekOfDays.length > 0) dates.push(weekOfDays); // 남은 날짜 추가

  const len = weekOfDays.length;
  if (len > 0 && len < 7) {
    for (let k = 1; k <= 7 - len; k += 1) {
      weekOfDays.push({
        day: k,
        year: year.value,
        month: month.value + 1,
      });
    }
  }
  return dates;
};

onBeforeMount(() => {
  currentYear.value = date.getFullYear();
  currentMonth.value = date.getMonth() + 1;
  year.value = currentYear.value;
  month.value = currentMonth.value;
});

onMounted(() => {
  calendarData();
});
</script>
