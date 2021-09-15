<template>
  <div>
    <h2>{{ msg }}</h2>
    <p>Ctrl + Click on canvas to add a Node. To link nodes: Ctrl + click to select source and mouseup to select target. Also drag and drop available</p>
    <!-- <button @click="addNode">New Node</button> -->
    <!--<button @click="addLink">New Link</button>-->
    <button @click="addInfo">New Info</button>
    <keep-alive>
      <NewChart :datashapes="datashapes" :datalinks="datalinks" />
    </keep-alive>
  </div>
</template>

<script>
import { defineAsyncComponent } from "vue";
import axios from "axios";
export default {
  name: "app",
  components: {
    NewChart: defineAsyncComponent(() => import("./components/NewChart.vue")),
  },
  data: function () {
    return {
      msg: "💃🕺",
      datashapes: [],
      datalinks: [],
      // newNode: {
      //   id: Math.floor(Math.random() * (100 - 9)) + 9,
      //   y: 0,
      //   x: 0,
      //   text: "example",
      //   index: null,
      // },
      // newLink: {
      //   text: "I am a link",
      //   source: Math.floor(Math.random() * (9 - 0)) + 0,
      //   target: Math.floor(Math.random() * (9 - 0)) + 0,
      // },
    };
  },
  created() {
    this.getShapes();
    this.getLinks();
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
    async getInfo() {
      let response = await axios.get("./Info.json");
      this.nodes = response.data;
    },

    // addNode() {
    //   this.datashapes = [...this.datashapes, this.newNode];
    // },
    // addLink() {
    //   this.datalinks = [...this.datalinks, this.newLink];
    // },
    addInfo() {
      alert("In progress");
      //this.datashapes= [...this.datashapes, this.nodes]
      //this.datalinks= [...this.datalinks, this.newLink]
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  text-align: center;
  margin-top: 2rem;
}
</style>
