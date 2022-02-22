<template>
  <div class="h-100 d-flex flex-column flex-grow-1 overflow-hidden">
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
          @event-clicked="emit('eventClicked', $event)"
          @day-pressed="(currentDay = $event), (mode = 'day')"
          @week-pressed="(currentDay = $event), (mode = 'week')"
          @month-pressed="(currentDay = currentMonth), (mode = 'month')"
          :month="currentMonth"
          controllable
          @previous="currentDay = currentDay.plus({ month: -1 })"
          @next="currentDay = currentDay.plus({ month: 1 })"
          :events="getEventsForMonth(currentMonth)"
        >
        </MonthView>
        <div class="m-2">
          <input type="text" class="form-control" placeholder="Termine Suchen..." v-model="filterQuery" />
        </div>
        <div class="p-1 ps-2" v-for="(group, index) of groups" :key="group.id" @click="group.checked = !group.checked">
          <input
            class="form-check-input"
            type="checkbox"
            v-model="group.checked"
            :style="{ backgroundColor: groupColors[index], borderColor: 'grey' }"
          />
          <label class="form-check-label ms-3" for="flexCheckDefault"> {{ group.name }}</label>
        </div>
      </div>
      <div class="d-flex flex-column flex-grow-1" v-if="mode == 'week' || mode == 'day'">
        <!-- date header -->
        <div class="flex-grow-1" style="display: flex; flex-direction: column">
          <div
            class="w-100 d-flex border-bottom"
            style="flex: 0 0 auto; padding-left: var(--time-axis-width)"
            :style="{ 'padding-right': weekViewScrollbarSize() + 'px' }"
          >
            <div
              v-for="date of currentWeek"
              :key="date.toISODate()"
              style="width: 0px; cursor: pointer"
              class="d-flex flex-grow-1 flex-column align-items-center"
              @click="(currentDay = date), (mode = 'day')"
            >
              <div class="text-muted mt-2" style="font-size: 14px" :class="getDayClasses(date, 'day')">{{ date.weekdayShort }}</div>
              <div class="h5 d-sm-none d-block fw-bold p-1 rounded-circle" :class="getDayClasses(date, 'num')">{{ date.day }}</div>
              <div class="h2 d-none d-sm-block fw-normal p-2 rounded-circle" :class="getDayClasses(date, 'num')">{{ date.day }}</div>
            </div>
          </div>
          <!-- date content -->
          <div class="overflow-auto d-flex flex-row" id="weekContainer" style="height: 0; flex: 1 1 auto; align-items: stretch">
            <div class="timeaxis-container" :style="{ '--num-of-hours': displayHours[1] - displayHours[0] }">
              <div v-for="num of displayHours[1] - displayHours[0] - 1" class="timeaxis" :key="num">
                {{ ("0" + (num + displayHours[0])).slice(-2) }}:00
              </div>
            </div>

            <div
              v-for="date of currentWeek"
              :style="{ '--num-of-hours': displayHours[1] - displayHours[0] }"
              :key="date.toISODate()"
              class="w-100 day-background position-relative overflow-hidden"
              @click="emit('timeClicked', date.plus({hours:Math.round(($event.offsetY/($event.target as any).offsetHeight*(displayHours[1] - displayHours[0])+displayHours[0])*4)/4}))"
            >
              <DayEvents
                @event-clicked="emit('eventClicked', $event)"
                :start="displayHours[0]"
                :end="displayHours[1]"
                :events="getEventsForDay(date)"
                :isToday="DateTime.now().startOf('day').equals(date)"
              ></DayEvents>
            </div>
          </div>
        </div>
      </div>
      <div class="d-flex flex-column flex-grow-1" v-else-if="mode == 'month'">
        <MonthView
          @event-clicked="emit('eventClicked', $event)"
          @day-pressed="(currentDay = $event), (mode = 'day')"
          @week-pressed="(currentDay = $event), (mode = 'week')"
          @month-pressed="(currentDay = currentMonth), (mode = 'month')"
          :month="currentMonth"
          :big="true"
          :events="getEventsForMonth(currentMonth)"
        >
        </MonthView>
      </div>
      <div class="d-flex flex-column flex-grow-1" v-else-if="mode == 'year'">
        <div class="overflow-auto" style="height: 0; flex: 1 1 auto; align-items: stretch">
          <div class="row mx-0" style="margin-top: -1.5rem">
            <MonthView
              @event-clicked="emit('eventClicked', $event)"
              @day-pressed="(currentDay = $event), (mode = 'day')"
              @week-pressed="(currentDay = $event), (mode = 'week')"
              @month-pressed="(currentDay = currentYear.plus({ month: month - 1 })), (mode = 'month')"
              class="col-12 col-md-6 col-lg-4 col-xxl-3 gy-4"
              v-for="month in 12"
              :key="month"
              :month="currentYear.plus({ month: month - 1 })"
              :events="getEventsForMonth(currentYear.plus({ month: month - 1 }))"
            >
            </MonthView>
          </div>
        </div>
      </div>
      <div class="d-flex flex-column flex-grow-1" v-else-if="mode == 'agenda'">
        <EventAgenda
          @event-clicked="emit('eventClicked', $event)"
          @day-pressed="(currentDay = $event), (mode = 'day')"
          :events="getFutureEvents(currentDay).slice(0, 25)"
        />
      </div>
    </div>
  </div>
</template>

<script lang="ts">
// eslint-disable-next-line @typescript-eslint/ban-ts-comment
//@ts-ignore
export type Event = {
  start: string;
  end: string;
  name: string;
  id: number;
  color?: string;
  group_id: number;
};

export type Group = {
  name: string;
  id: number;
  checked: boolean;
  color?: string;
};

export type Layout = {
  groupSize: number;
  groupIndex: number;
};

import { defineComponent } from "vue";
export default defineComponent({
  name: "Calendar",
});
</script>

<script lang="ts" setup>
import { DateTime } from "luxon";

import { computed, defineProps, ref, toRefs, defineEmits } from "vue";
import ButtonGroup from "./ButtonGroup.vue";
import MonthView from "./MonthView.vue";
import DayEvents from "./DayEvents.vue";
import EventAgenda from "./EventAgenda.vue";
function log(a: any) {
  console.log(a);
}
const props = defineProps<{
  displayHours: [number, number];
  groups: Group[];
  events: Event[];
}>();
const { events, groups: groupsProp, displayHours } = toRefs(props);

const emit = defineEmits<{
  (e: "update:groups", value: Group[]): void;
  (e: "eventClicked", value: Event): void;
  (e: "timeClicked", value: DateTime): void;
}>();

let groups = computed({ get: () => groupsProp.value, set: (groups: Group[]) => emit("update:groups", groups) });

const isMobile = window.innerWidth < 576;

const mode = ref<"week" | "day" | "month" | "year" | "agenda">(isMobile ? "day" : "month");
const timeFrame = computed(() => (mode.value == "agenda" ? "day" : mode.value));

const currentDay = ref(DateTime.now());

const getDayClasses = (date: DateTime, type: "num" | "day") => {
  return {
    "is-weekend": date.weekday == 6 || date.weekday == 7,
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

const groupColors = computed(() => {
  const genRndColor = (i: number) =>
    [50, 25, 80].flatMap((l) => [
      `hsl(0, 100%, ${l}%)`,
      `hsl(127, 80%, ${l}%)`,
      `hsl(241, 100%, ${l}%)`,
      `hsl(35, 100%, ${l}%)`,
      `hsl(181, 100%, ${l}%)`,
      `hsl(300, 100%, ${l}%)`,
      `hsl(58, 80%, ${l}%)`,
    ])[i] || "#" + ("000000" + Math.floor((Math.sin(i - 20) * 10000 - Math.floor(Math.sin(i - 20) * 10000)) * 0xffffff).toString(16)).slice(-6);

  let rndColorIndex = 0;
  return groups.value.map((g) => g.color || genRndColor(rndColorIndex++));
});

const eventsWithColor = computed(() =>
  events.value.map((e) => ({ ...e, color: e.color || groupColors.value[groups.value.findIndex((g) => g.id == e.group_id)] }))
);

const filterQuery = ref("");

const filteredEvents = computed(() =>
  eventsWithColor.value
    .filter((e) => groups.value.filter((g) => g.checked).some((g) => g.id == e.group_id))
    .filter((e) => e.name.includes(filterQuery.value))
);

const getEventsForDay = (day: DateTime) =>
  filteredEvents.value
    .filter((e) => day.toFormat("yyyy-LL-dd") == e.start.split("T")[0])
    .map((e) => ({
      ...e,
      start: e.start.split("T")[1],
      end: e.end.split("T")[1],
    }));

const getFutureEvents = (day: DateTime) =>
  filteredEvents.value.filter((e) => day.startOf("day") <= DateTime.fromFormat(e.start.split("T")[0], "yyyy-LL-dd").startOf("day"));

const getEventsForMonth = (day: DateTime) =>
  filteredEvents.value.filter((e) => day.startOf("month").equals(DateTime.fromFormat(e.start.split("T")[0], "yyyy-LL-dd").startOf("month")));

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
  --hour-height: 63px;
  --half-hour-height: calc(var(--hour-height) / 2);
  --day-start-offset: 0px;

  --dayview-border-color: #ddd;
  --dayview-background-color: #fff;
  --dayview-half-hour-line-color: #f0f0f0;
}
.day-background {
  height: calc(var(--hour-height) * var(--num-of-hours));
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
  height: calc(var(--hour-height) * var(--num-of-hours));
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
