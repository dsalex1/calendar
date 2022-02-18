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
      {{ month.toFormat("LLLL yyyy") }}
      <div v-if="controllable" class="btn" @click="emit('next')">&gt;</div>
    </div>
    <div v-if="!big && !controllable" class="d-flex mb-2 align-items-center" style="font-size: 20px">
      {{ month.toFormat("LLLL") }}
    </div>

    <div class="d-flex flex-row">
      <div class="day" v-if="!big"></div>
      <div class="day" :class="{ colBase: big }" v-for="day of weekdayHeader" :key="day">
        <div>{{ day }}</div>
      </div>
    </div>

    <template v-if="big">
      <div v-for="row of 6" :key="row" class="d-flex flex-row flex-grow-1 border-top">
        <div
          v-for="(day, index) of Array(7)
            .fill(null)
            .map((_, i) => month.startOf('week').plus({ week: row - 1, day: i }))"
          :key="index"
          class="colBase border-start h-100 d-flex justify-content-between px-2"
          :class="{
            dayOutOfMonth: day.month != month.month,
          }"
        >
          <div class="fw-bold">{{ index == 0 ? day.weekNumber : "" }}</div>
          <div>
            {{ day.day }}
            {{
              events
                .filter((e) => day.startOf("day").equals(DateTime.fromFormat(e.start.split("T")[0], "yyyy-LL-dd").startOf("day")))
                .map((e) => e.name)
                .join(", ")
            }}
          </div>
        </div>
      </div>
    </template>

    <template v-if="!big">
      <div v-for="row of 6" :key="row" class="d-flex flex-row">
        <div class="day weeknumber">
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
          class="day"
          :class="{
            dayOutOfMonth: day.month != month.month,
          }"
        >
          {{ day.day }}
        </div>
      </div>
    </template>
  </div>
</template>

<script lang="ts" setup>
import { DateTime } from "luxon";

import { defineProps, toRefs, defineEmits, computed } from "vue";

type Event = {
  start: string;
  end: string;
  name: string;
  id: number;
  color: string;
};

const props = defineProps<{
  month: DateTime;
  controllable?: boolean;
  big?: boolean;
  events: Event[];
}>();
const { events, month, big, controllable } = toRefs(props);

const emit = defineEmits(["previous", "next"]);

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
</script>

<style scoped lang="scss">
.day {
  width: 30px;
  height: 30px;
  display: flex;
  align-items: center;
  justify-content: center;
}
.weeknumber {
  background-color: #e7eaeb;
}
.dayOutOfMonth {
  color: #d0d3d4;
}
.colBase {
  flex-basis: calc(100% / 7);
}
</style>
