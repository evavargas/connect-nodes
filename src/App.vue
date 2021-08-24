<template>
  <div>
     <div>
       <keep-alive>
      <NewChart :shapes="shapes" :links="links" />
      </keep-alive>
    </div>
    <h1>Nodes linked with D3</h1>
    <h2>{{ msg }}</h2>
    <div>
      <Chart :dotsData="nodes" />
    </div>
  </div>
</template>

<script>
import { defineAsyncComponent } from 'vue'
import axios from "axios";
import Chart from "./components/Chart.vue";
export default {
  name: "app",
  components: {
    NewChart: defineAsyncComponent(() =>
      import('./components/NewChart.vue')
    ),
    Chart,
  },
  data: function () {
    return {
      msg: "💃🕺",
      nodes: [],
      shapes: [],
      links: [],
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
      this.shapes = response.data;
    },
    async getLinks() {
      let response = await axios.get("./links.json");
      this.links = response.data;
    },
    async getNodes() {
      let response = await axios.get("./dots.json");
      this.nodes = response.data;
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
.btn-addnode{
  background-color: greenyellow;
  width: 4rem;
  border-color: hotpink;
  border-style: solid;
  border-width: 1px;
}
.form-addnode{
  width: 8rem;
}
.form-addnode input {
  width: 6rem;
}
</style>
