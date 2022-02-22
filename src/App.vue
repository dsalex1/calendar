<template>
  <div class="container pt-5">
    <div class="card">
      <h5 class="card-header">calendar</h5>
      <div class="card-body">
        <div style="min-height: 80vh" class="d-flex flex-column">
          <Calendar
            msg="Calendar"
            :displayHours="[7, 18]"
            :events="events"
            v-model:groups="groups"
            @eventClicked="events = events.filter((e) => e.id != $event.id)"
            @timeClicked="
              events.push({ start: $event.toISO(), end: $event.plus({ hours: 1 }).toISO(), name: $event.toISO(), id: Math.random(), group_id: 1 })
            "
          />
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { ref } from "vue";
import Calendar, { Event, Group } from "./components/Calendar.vue";

const events = ref<Event[]>([
  { start: "2022-03-16T17:00", end: "2022-03-16T19:00", name: "16.03 17 uhr", id: 1, group_id: 7 },
  { start: "2022-03-16T17:00", end: "2022-03-16T18:00", name: "fsdkjhfs", id: 2, group_id: 8 },
  { start: "2022-03-16T08:30", end: "2022-03-16T10:30", name: "udkauhd", id: 3, group_id: 9 },
  { start: "2022-03-16T10:30", end: "2022-03-16T11:30", name: "jfd", id: 4, group_id: 1 },
  { start: "2022-03-16T09:45", end: "2022-03-16T10:30", name: "jfd", id: 5, group_id: 1 },
  { start: "2022-03-15T10:30", end: "2022-03-15T11:30", name: "15.03 10 uhr", id: 6, group_id: 2 },
  { start: "2022-03-15T09:45", end: "2022-03-15T10:30", name: "jfd", id: 7, group_id: 3 },
  { start: "2022-03-17T17:00", end: "2022-03-17T19:00", name: "fsdkjhfs", id: 8, group_id: 2 },
  { start: "2022-03-17T17:00", end: "2022-03-17T18:00", name: "fsdkjhfs", id: 9, group_id: 3 },
  { start: "2022-03-17T08:30", end: "2022-03-17T10:30", name: "udkauhd", id: 10, group_id: 4 },
  { start: "2022-03-17T08:30", end: "2022-03-17T10:30", name: "udkauhd", id: 11, group_id: 4 },
  { start: "2022-03-17T08:30", end: "2022-03-17T10:30", name: "udkauhd", id: 12, group_id: 4 },
  { start: "2022-03-17T08:30", end: "2022-03-17T10:30", name: "udkauhd", id: 13, group_id: 4 },
  { start: "2022-03-17T08:30", end: "2022-03-17T10:30", name: "udkauhd", id: 14, group_id: 4 },
  { start: "2022-03-17T08:30", end: "2022-03-17T10:30", name: "udkauhd", id: 15, group_id: 4 },
  { start: "2022-03-17T08:30", end: "2022-03-17T10:30", name: "udkauhd", id: 16, group_id: 4 },
  { start: "2022-03-17T08:30", end: "2022-03-17T10:30", name: "udkauhd", id: 17, group_id: 4 },
  { start: "2022-03-17T08:30", end: "2022-03-17T10:30", name: "udkauhd", id: 18, group_id: 4 },
  { start: "2022-03-17T08:30", end: "2022-03-17T10:30", name: "udkauhd", id: 19, group_id: 4 },
  { start: "2022-03-17T08:30", end: "2022-03-17T10:30", name: "udkauhd", id: 20, group_id: 4 },
  { start: "2022-03-17T10:30", end: "2022-03-17T11:30", name: "jfd", id: 21, group_id: 5 },
  { start: "2022-03-17T09:45", end: "2022-03-17T10:30", name: "jfd", id: 22, group_id: 6 },
  { start: "2022-03-17T17:00", end: "2022-03-17T19:00", name: "fsdkjhfs", id: 23, group_id: 1 },
  { start: "2022-03-18T07:00", end: "2022-03-18T08:00", name: "fsdkjhfs", id: 24, group_id: 2, color: "black" },
  { start: "2022-03-18T08:30", end: "2022-03-18T10:30", name: "udkauhd", id: 25, group_id: 3 },
  { start: "2022-03-18T10:30", end: "2022-03-18T11:30", name: "jfd", id: 26, group_id: 4 },
  { start: "2022-03-18T09:45", end: "2022-03-18T10:30", name: "jfd", id: 27, group_id: 5 },
  { start: "2022-03-19T09:45", end: "2022-03-19T11:30", name: "jfd", id: 28, group_id: 6 },
  { start: "2022-03-19T05:45", end: "2022-03-19T18:30", name: "jfd", id: 29, group_id: 7 },
  { start: "2022-03-19T13:00", end: "2022-03-19T14:00", name: "jfd", id: 33, group_id: 8 },
  { start: "2022-03-19T13:00", end: "2022-03-19T14:00", name: "jfd", id: 34, group_id: 9 },
  { start: "2022-03-20T13:00", end: "2022-03-20T14:00", name: "jfd", id: 35, group_id: 0 },
  { start: "2022-03-20T13:00", end: "2022-03-20T14:00", name: "jfd", id: 36, group_id: 1 },
]);

const groups = ref<Group[]>([
  { id: 1, name: "Group1", checked: true },
  { id: 2, name: "Group2", checked: true, color: "black" },
  { id: 3, name: "Group3", checked: true },
  { id: 4, name: "Group4", checked: true },
  { id: 5, name: "Group5", checked: true },
  { id: 6, name: "Group6", checked: true },
  { id: 7, name: "Group3", checked: true },
  { id: 8, name: "Group4", checked: true },
  { id: 9, name: "Group5", checked: true },
]);
</script>
<style lang="scss"></style>
