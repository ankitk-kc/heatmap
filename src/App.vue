<template>
  <div>
    <HeatmapCalendarVue @ChangeOnDate="ChangeOnDate" />
    <ChangesDoneVue :changesDone="changesDone" />
  </div>

</template>

<script lang="ts">
import { defineComponent, ref } from 'vue';
import HeatmapCalendarVue from './components/HeatmapCalendar.vue';
import ChangesDoneVue from './components/ChangesDone.vue';

export default defineComponent({
  name: 'App',
  components: {
    HeatmapCalendarVue,
    ChangesDoneVue
  },
  setup() {
    const changesDone = ref('')
    function convertDate(inputFormat: Date) {
      function pad(s: number) { return (s < 10) ? '0' + s : s; }
      var d = new Date(inputFormat)
      return [pad(d.getDate()), pad(d.getMonth() + 1), d.getFullYear()].join('-')
    }

    function ChangeOnDate(arg: string) {
      const date = new Date(arg)
      // changesDone.value = "ankit pal"
      changesDone.value = convertDate(date)
    }


    return { ChangeOnDate, changesDone }
  }
});
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
