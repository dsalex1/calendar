<template>
  <div
    :class="{
      w100: big,
      'd-flex': big,
      'flex-column': big,
      'flex-grow-1': big,
    }"
  >
    <div v-if="!big && controllable" class="d-flex mb-2 align-items-center justify-content-between" style="font-size: 18px">
      <div v-if="controllable" class="btn" @click="emit('previous')">&lt;</div>
      <div style="cursor: pointer" @click="emit('monthPressed')">{{ month.toFormat("LLLL yyyy") }}</div>
      <div v-if="controllable" class="btn" @click="emit('next')">&gt;</div>
    </div>
    <div v-if="!big && !controllable" class="d-flex mb-2 align-items-center" style="font-size: 20px; cursor: pointer" @click="emit('monthPressed')">
      {{ month.toFormat("LLLL") }}
    </div>

    <div class="d-flex flex-row">
      <div class="day" v-if="!big"></div>
      <div class="day" :class="{ 'flex-base-equal': big }" v-for="day of weekdayHeader" :key="day">
        <div>{{ day }}</div>
      </div>
    </div>

    <template v-if="big">
      <div v-for="row of 6" :key="row" class="d-flex flex-row flex-grow-1 border-top flex-base-equal">
        <div
          v-for="(day, index) of Array(7)
            .fill(null)
            .map((_, i) => month.startOf('week').plus({ week: row - 1, day: i }))"
          :key="index"
          class="flex-grow-1 border-start h-100 px-1 px-sm-"
          style="width: 0"
          :class="{
            dayOutOfMonth: day.month != month.month,
          }"
        >
          <div class="d-flex justify-content-between">
            <div class="fw-bold" style="cursor: pointer" @click="emit('weekPressed', day)">{{ index == 0 ? day.weekNumber : "" }}</div>
            <div style="cursor: pointer" @click="emit('dayPressed', day)" :class="{ isToday: DateTime.now().startOf('day').equals(day) }">
              {{ day.day }}
            </div>
          </div>
          <div
            v-for="event of getDayEvents(day).slice(0, 4)"
            :key="event.id"
            style="white-space: nowrap; text-overflow: ellipsis; overflow: hidden; cursor: pointer"
            @click="emit('eventClicked', event)"
          >
            <div class="custom-bullets" :style="'--bullet-color:' + event.color"></div>
            {{ event.name }}
          </div>
          <div style="white-space: nowrap; overflow: hidden" v-if="getDayEvents(day).length > 4">+{{ getDayEvents(day).length - 4 }} weitere</div>
        </div>
      </div>
    </template>

    <template v-if="!big">
      <div v-for="row of 6" :key="row" class="d-flex flex-row">
        <div class="day weeknumber" style="cursor: pointer" @click="emit('weekPressed', month.startOf('week').plus({ week: row - 1 }))">
          {{
            month
              .startOf("week")
              .plus({ week: row - 1 })
              .toFormat("W")
          }}
        </div>
        <div
          v-for="(day, key) of Array(7)
            .fill(null)
            .map((_, i) => month.startOf('week').plus({ week: row - 1, day: i }))"
          :key="key"
          class="day d-flex flex-column"
          style="cursor: pointer"
          :class="{
            dayOutOfMonth: day.month != month.month,
            isToday: DateTime.now().startOf('day').equals(day),
          }"
          @click="emit('dayPressed', day)"
        >
          {{ day.day }}
          <div class="rounded-circle" style="width: 5px; height: 5px; margin-top: -5px" :style="'background-color:' + getOccupancyColor(day)"></div>
        </div>
      </div>
    </template>
  </div>
</template>

<script lang="ts" setup>
import { DateTime } from "luxon";

import { defineProps, toRefs, defineEmits, computed } from "vue";
import type { Event } from "./Calendar.vue";

const props = defineProps<{
  month: DateTime;
  controllable?: boolean;
  big?: boolean;
  events: Event[];
}>();
const { events, month, big, controllable } = toRefs(props);

const emit = defineEmits<{
  (e: "previous"): void;
  (e: "next"): void;
  (e: "dayPressed", value: DateTime): void;
  (e: "weekPressed", value: DateTime): void;
  (e: "monthPressed"): void;
  (e: "eventClicked", value: Event): void;
}>();

const weekdayHeader = computed(() =>
  Array(7)
    .fill(null)
    .map((_, i) =>
      DateTime.now()
        .startOf("week")
        .plus({ days: i })
        .toFormat(big?.value ? "ccc" : "ccccc")
    )
);

function getDayEvents(date: DateTime) {
  return events.value.filter((e) => date.startOf("day").equals(DateTime.fromFormat(e.start.split("T")[0], "yyyy-LL-dd").startOf("day")));
}
function getOccupancyColor(date: DateTime) {
  const number = getDayEvents(date).length;
  if (number == 0) return "transparent";
  if (number < 3) return "hsl(59, 85%, 50%)";
  if (number < 10) return "orange";
  return "red";
}
</script>

<style scoped lang="scss">
.day {
  width: 30px;
  height: 30px;
  display: flex;
  align-items: center;
  justify-content: center;
}
.isToday {
  background-color: #b5d2ec;
  border-radius: 50%;
}
.weeknumber {
  background-color: #e7eaeb;
}
.dayOutOfMonth {
  color: #d0d3d4;
}
.flex-base-equal {
  flex-basis: 100%;
}
.custom-bullets {
  content: "";
  height: 0.6em;
  width: 0.6em;
  border-radius: 50%;
  background-color: var(--bullet-color);
  display: inline-block;
}
</style>
