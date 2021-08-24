<template>
  <div>
     <div>
       <button @click="addData">New Node</button>
       <keep-alive>
       <NewChart :datashapes="datashapes" :datalinks="datalinks" />
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
import { defineAsyncComponent, reactive } from 'vue'
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
      newNode: {
        id: 0,
        y: 30,
        x: 30,
        text: "hola",
        vy: 4,
        vx: 4,
        index:0
      },

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
    addData() {
      console.log(this.datashapes);
      let newNode = this.newNode;
      newNode.id = this.datashapes.length;
      newNode.index = newNode.id;
      console.log(newNode);
      this.datashapes.push(newNode);
      console.log(this.datashapes);
    },
  },
    computed: {
    datashapes: {
      get() {
        return this.shapes;
      },
      set(newValue) {
        this.concat(newValue);
      },
    },
    datalinks: {
      get() {
        return reactive(this.links);
      },
      set(newValue) {
        this.links.concat(newValue);
      },
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
