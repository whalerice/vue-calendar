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
          <td v-for="(item, index2) in row" :key="index2">
            <div class="date-panel">
              <span
                class="date"
                :class="{
                  'text-prev': item.month !== month,
                  today:
                    today.day === item.day &&
                    today.year === item.year &&
                    today.month === item.month,
                }"
              >
                {{ item.day }}
              </span>
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
import calendar from 'dayjs/plugin/calendar';
import { onBeforeMount, onMounted, ref } from 'vue';
dayjs.extend(calendar);

interface IDate {
  day: number | null;
  year?: number;
  month?: number;
}

const today = ref<IDate>({ day: 0 });
const date = new Date();
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

// const lastMonthStart = ref<number>(0);
// const nextMonthStart = ref<number>(0);

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

  console.log(monthLastDate);

  const dates: any = [];
  let weekOfDays = [];
  while (day <= monthLastDate) {
    if (day === 1) {
      // 1일이 어느 요일인지에 따라 테이블에 그리기 위한 지난 셀의 날짜들을 구할 필요가 있다.
      for (let j = 0; j < monthFirstDay; j += 1) {
        //if (j === 0) lastMonthStart.value = prevDay; // 지난 달에서 제일 작은 날짜
        const item = {
          day: prevDay,
          year,
          month: month.value - 1,
        };
        weekOfDays.push(item);
        prevDay += 1;
      }
    }

    weekOfDays.push({ day, year, month });

    if (weekOfDays.length === 7) {
      // 일주일 채우면
      // if (weeks.value.length > 7) weekOfDays.push(-1);
      dates.push(weekOfDays);
      weekOfDays = []; // 초기화
    }
    day += 1;
  }
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
  // if (weeks.value.length > 7) weekOfDays.push(-2);
  if (weekOfDays.length > 0) dates.push(weekOfDays); // 남은 날짜 추가
  //nextMonthStart.value = weekOfDays[0]; // 이번 달 마지막 주에서 제일 작은 날짜
  return dates;
};

onBeforeMount(() => {
  currentYear.value = date.getFullYear();
  currentMonth.value = date.getMonth() + 1;
  year.value = currentYear.value;
  month.value = currentMonth.value;
  today.value = { day: date.getDate(), year: currentYear.value, month: currentMonth.value };
});

onMounted(() => {
  calendarData();
});
</script>
