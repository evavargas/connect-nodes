<template>
  <div>
    <div>
      <button @click="addData">New Node</button>
      <keep-alive>
        <NewChart :datashapes="datashapes" :datalinks="datalinks" />
      </keep-alive>
    </div>
    <!-- <Graph :datashapes="datashapes" :datalinks="datalinks" /> -->
    <h1>Nodes linked with D3</h1>
    <h2>{{ msg }}</h2>
    <div>
      <Chart :dotsData="nodes" />
    </div>
  </div>
</template>

<script>
import { defineAsyncComponent } from "vue";
import axios from "axios";
import Chart from "./components/Chart.vue";
//import Graph from "./components/Graph.vue";
export default {
  name: "app",
  components: {
    NewChart: defineAsyncComponent(() => import("./components/NewChart.vue")),
    Chart,
    //Graph,
  },
  data: function () {
    return {
      msg: "💃🕺",
      nodes: [],
      datashapes: [],
      datalinks: [],
      newNode: { id: 9, y: 10, x: 10, text: "hola", vy: 0, vx: 0, index:null },
      newLink: { text: "I am alink", source: 9, target: 1 },
    };
  },
  created() {
    this.getShapes();
    this.getLinks();
    this.getNodes();
  },

  methods: {
    async getShapes() {
      let response = await axios.get("./shapes.json");
      this.datashapes = response.data;
    },
    async getLinks() {
      let response = await axios.get("./links.json");
      this.datalinks = response.data;
    },
    async getNodes() {
      let response = await axios.get("./dots.json");
      this.nodes = response.data;
    },
    addData() {
      this.datashapes= [...this.datashapes, this.newNode]
      this.datalinks= [...this.datalinks, this.newLink]
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
  margin-top: 2rem;
}
.btn-addnode {
  background-color: greenyellow;
  width: 4rem;
  border-color: hotpink;
  border-style: solid;
  border-width: 1px;
}
.form-addnode {
  width: 8rem;
}
.form-addnode input {
  width: 6rem;
}
</style>
