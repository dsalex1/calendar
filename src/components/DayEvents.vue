<template>
  <div
    v-for="event in layoutedEvents"
    :key="JSON.stringify(event)"
    class="alert alert-primary p-0 border-0 rounded-0 eventCard"
    :style="getEventStyle(event)"
    draggable="true"
  >
    <div class="eventCardContent">
      <div class="ps-2">{{ event.name }}</div>
    </div>
  </div>
</template>

<script lang="ts">
// eslint-disable-next-line @typescript-eslint/ban-ts-comment
//@ts-ignore
type Event = {
  start: string;
  end: string;
  name: string;
  id: number;
  color: string;
};

type Layout = {
  groupSize: number;
  groupIndex: number;
};
</script>

<script lang="ts" setup>
import { computed, defineProps, toRefs } from "vue";

const props = defineProps<{
  events: Event[];
}>();
const { events } = toRefs(props);

const layoutedEvents = computed(() => {
  function collidesWith(a: Event, b: Event) {
    return a.end > b.start && a.start < b.end;
  }

  let groups: Event[][] = [];
  for (let event of events.value) {
    //find groups where it overlaps in
    let groupsCollisions = groups.filter((group) => group.some((e) => collidesWith(e, event)));

    //if we found at least one combine the grups and push it there, else create new group
    if (groupsCollisions.length > 0) {
      let newGroup = [...groupsCollisions.flat(), event];

      for (let group of groupsCollisions) group.splice(0, group.length);

      groupsCollisions[0].splice(0, groupsCollisions.length, ...newGroup);
    } else groups.push([event]);
  }

  return groups.flatMap((events) => {
    let columns: Event[][] = [];
    for (let event of events) {
      //find first column where event fits
      let firstEmptyColumn = columns.find((column) => column.every((e) => !collidesWith(e, event)));
      //if we found one push it there, else create new column
      if (firstEmptyColumn) firstEmptyColumn.push(event);
      else columns.push([event]);
    }
    return columns.map((column, index) => column.map((e, i) => ({ ...e, groupIndex: index, groupSize: columns.length }))).flat();
  });
});

function getHours(time: string) {
  return parseInt(time.split(":")[0]) + parseInt(time.split(":")[1]) / 60;
}

function getEventStyle(event: Event & Layout) {
  let start = getHours(event.start);
  let end = getHours(event.end);

  return {
    top: `calc(${start} / 24 * 100%)`,
    left: `calc(calc(100% - 5px) / ${event.groupSize / event.groupIndex})`,
    width: `calc(calc(100% - 5px) / ${event.groupSize})`,
    height: `calc(${end - start} / 24 * 100%)`,
    position: "absolute",
    fontSize: "10px",
    backgroundColor: event.color,
  };
}
</script>

<style scoped lang="scss">
.eventCard {
  [draggable="true"] {
    cursor: move;
  }
  color: rgb(71, 71, 71);
  box-shadow: #1a090940 -2px 1px 3px, #aaaaaa40 0 -1px 3px;
}
.eventCardContent {
  height: 100%;
  margin: 0 0 0 4px;
  background-color: #ffffffe5;
  display: flex;
  flex-direction: column;
  &:hover {
    background-color: #ffffffa1;
  }
  transition: 0.15s;
}
</style>
