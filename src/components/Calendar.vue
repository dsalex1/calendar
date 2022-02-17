<template>
  <div class="h-100 d-flex flex-column">
    <!-- menu bar -->
    <div class="bg-light d-flex justify-content-between text-center p-2 border-bottom flex-wrap pb-0">
      <div class="d-flex mb-2">
        <a class="btn btn-secondary me-3" data-bs-toggle="collapse" href="#sideControls"> ≡ </a>
        <button class="btn btn-outline-secondary d-none me-2 d-sm-block" @click="currentDay = DateTime.now()">Heute</button>
        <div class="btn-group me-3">
          <button class="btn btn-outline-secondary" @click="currentDay = currentDay.plus({ [timeFrame]: -1 })">&lt;</button>
          <button class="btn btn-outline-secondary" @click="currentDay = currentDay.plus({ [timeFrame]: 1 })">&gt;</button>
        </div>
        <div class="d-none d-sm-flex align-items-center" style="font-size: 20px">{{ currentDayReadable }}</div>
      </div>
      <div class="d-none d-sm-block mb-2">
        <ButtonGroup
          v-model="mode"
          :options="{ day: 'Tag', week: 'Woche', month: 'Monat', year: 'Jahr', agenda: 'Agenda' }"
          name="type"
        ></ButtonGroup>
      </div>
      <div class="d-block d-sm-none mb-2">
        <ButtonGroup
          v-model="mode"
          class="btn-sm"
          :options="{ day: 'T.', week: 'W.', month: 'M.', year: 'J.', agenda: 'A.' }"
          name="type"
        ></ButtonGroup>
      </div>
    </div>
    <div class="d-flex flex-row flex-grow-1">
      <!-- side controls -->
      <div class="bg-light collapse hide" :class="{ show: !isMobile }" id="sideControls">
        <MonthView
          :month="currentMonth"
          controllable
          @previous="currentDay = currentDay.plus({ month: -1 })"
          @next="currentDay = currentDay.plus({ month: 1 })"
        >
        </MonthView>
      </div>
      <div class="d-flex flex-column flex-grow-1" v-if="mode == 'week' || mode == 'day'">
        <!-- date header -->
        <div class="flex-grow-1" style="display: flex; flex-direction: column">
          <div
            class="w-100 d-flex border-bottom"
            style="flex: 0 0 auto; padding-left: var(--time-axis-width)"
            :style="{ 'padding-right': weekViewScrollbarSize() + 'px' }"
          >
            <div v-for="date of currentWeek" :key="date.toISODate()" style="width: 0px" class="d-flex flex-grow-1 flex-column align-items-center">
              <div class="text-muted mt-2" style="font-size: 14px" :class="getDayClasses(date, 'day')">{{ date.weekdayShort }}</div>
              <div class="h5 d-sm-none d-block fw-bold p-1 rounded-circle" :class="getDayClasses(date, 'num')">{{ date.day }}</div>
              <div class="h2 d-none d-sm-block fw-normal p-2 rounded-circle" :class="getDayClasses(date, 'num')">{{ date.day }}</div>
            </div>
          </div>
          <!-- date content -->
          <div class="overflow-auto d-flex flex-row" id="weekContainer" style="height: 0; flex: 1 1 auto; align-items: stretch">
            <div class="timeaxis-container">
              <div v-for="num of 23" class="timeaxis" :key="num">{{ ("0" + num).slice(-2) }}:00</div>
            </div>

            <div v-for="date of currentWeek" :key="date.toISODate()" class="w-100 day-background"></div>
          </div>
        </div>
      </div>
      <div class="d-flex flex-column flex-grow-1" v-else-if="mode == 'month'">
        <MonthView :month="currentMonth" :big="true"> </MonthView>
      </div>
      <div class="d-flex flex-column flex-grow-1" v-else-if="mode == 'year'">
        <div class="overflow-auto" style="height: 0; flex: 1 1 auto; align-items: stretch">
          <div>
            <div class="row mx-0">
              <MonthView
                class="col-12 col-md-6 col-lg-4 col-xxl-3 gy-5"
                v-for="month in 12"
                :key="month"
                :month="currentYear.plus({ month: month - 1 })"
              >
              </MonthView>
            </div>
          </div>
        </div>
      </div>
      <div class="d-flex flex-column flex-grow-1" v-else-if="mode == 'agenda'">agenda</div>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { DateTime } from "luxon";

import { computed, defineProps, ref, toRefs } from "vue";
import ButtonGroup from "./ButtonGroup.vue";
import MonthView from "./MonthView.vue";

const isMobile = window.innerWidth < 576;

const mode = ref<"week" | "day" | "month" | "year" | "agenda">(isMobile ? "day" : "month");
const timeFrame = computed(() => (mode.value == "agenda" ? "day" : mode.value));

const currentDay = ref(DateTime.now());

const getDayClasses = (date: DateTime, type: "num" | "day") => {
  return {
    "is-weekend": date.weekday == 1 || date.weekday == 7,
    "is-today": date.startOf("day").equals(DateTime.now().startOf("day")) && type == "num",
    "is-current": date.startOf("day").equals(currentDay.value.startOf("day")) && type == "num",
  };
};

const currentWeek = computed(() =>
  Array({ day: 1, week: 7, month: null, year: null }[timeFrame.value])
    .fill(null)
    .map((_, i) => currentDay.value.startOf(timeFrame.value).plus({ days: i }))
);

const currentMonth = computed(() => currentDay.value.startOf("month"));
const currentYear = computed(() => currentDay.value.startOf("year"));

const currentDayReadable = computed(() =>
  currentDay.value.toFormat({ day: "dd. LLLL, yyyy", week: "LLLL, yyyy", month: "LLLL, yyyy", year: "yyyy" }[timeFrame.value])
);

const weekViewScrollbarSize = () => (/Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(navigator.userAgent) ? 1 : 17);
</script>

<style scoped lang="scss">
.is-weekend {
  color: rgb(229 63 44) !important;
}
.is-current {
  /* to highlight current looks bad*/
}
.is-today {
  background-color: #b5d2ec;
}

div {
  --time-axis-width: 2.5em;
  --day-length: 24;
  --day-height: 1008px;
  --hour-height: 42px;
  --half-hour-height: 20.5px;
  --day-start-offset: 0px;

  --dayview-border-color: #ddd;
  --dayview-background-color: #fff;
  --dayview-half-hour-line-color: #f0f0f0;
}
.day-background {
  height: var(--day-height);
  border-left: 1px solid #ddd;
  background-image: linear-gradient(
    to bottom,
    transparent,
    transparent var(--half-hour-height),
    var(--dayview-half-hour-line-color) 1px,
    transparent calc(var(--half-hour-height) + 1px),
    transparent calc(var(--hour-height) - 1px),
    var(--dayview-border-color) 1px
  );
  background-size: 100% var(--hour-height);
}
.timeaxis {
  display: flex;
  flex: 0 0 var(--hour-height);
  align-items: center;
  justify-content: flex-end;
  padding: 0.6em 1em 0;
  font-size: 0.7em;
  color: #b0b0b0;
  white-space: nowrap;
}
.timeaxis-container {
  height: var(--day-height);
  display: flex;
  flex-flow: column nowrap;
  overflow: hidden;
  background-position-x: 100%;
  background-repeat: repeat-y;
  flex: 0 0 var(--time-axis-width);
  background-image: linear-gradient(to bottom, transparent calc(var(--hour-height) - 1px), var(--dayview-border-color) 1px);
  background-size: 0.5em var(--hour-height);
  background-position-y: var(--day-start-offset);
}
.timeaxis-container {
  padding-top: calc(var(--hour-height) / 2 - 3px);
}
</style>
