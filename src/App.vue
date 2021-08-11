<template>
  <div>
    <!-- <img alt="Vue logo" src="./assets/logo.png" /> -->
    <!-- <HelloWorld msg="Welcome to Your Vue.js App" /> -->
    <Canvas />
    <!-- <NestChart :tweetData="loadData" /> -->
    <Chart :dotsData="entryData" />
  </div>
</template>

<script>
import * as d3 from "d3";
//import HelloWorld from "./components/HelloWorld.vue";
import Chart from "./components/Chart.vue";
//import NestChart from "./components/NestChart.vue";
//import Canvas from "./components/Canvas.vue";

export default {
  name: "app",
  components: {
    //  HelloWorld,
    Chart,
    // NestChart,
    //Canvas,
  },
  data: function () {
    return {
      loadData: [],
      entryData: [],
    };
  },
  mounted() {
//    this.fetchData();
    this.fetchEntryData();
  },
  methods: {
    // async fetchData() {
    //   let data = await d3.json("./tweets.json");
    //   this.loadData = data;
    // },
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
