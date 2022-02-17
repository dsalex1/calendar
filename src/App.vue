<template>
  <div class="container pt-5">
    <div class="card">
      <h5 class="card-header">calendar</h5>
      <div class="card-body">
        <div style="height: 80vh">
          <Calendar msg="Calendar" />
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import Calendar from "./components/Calendar.vue";

export default defineComponent({
  name: "App",
  components: {
    Calendar,
  },
});
</script>

<style lang="scss"></style>
