<template>
  <div>
    <Chart :dotsData="entryData" />
  </div>
</template>

<script>
import * as d3 from "d3";
import Chart from "./components/Chart.vue";
export default {
  name: "app",
  components: {
    Chart,
  },
  data: function () {
    return {
      entryData: [],
    };
  },
  created() {
    this.fetchEntryData();
  },
  methods: {

    async fetchEntryData() {
      let data = await d3.json("./dots.json");
      this.entryData = data;
    },
  },
};
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
