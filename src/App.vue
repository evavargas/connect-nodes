<template>
  <div>
    <h2>{{ msg }}</h2>
    
    <button @click="addInfo">New Info</button>
    <keep-alive>
      <NewChart :datashapes="datashapes" :datalinks="datalinks" />
    </keep-alive>
    <p class="message">To add a node: (1) right click. (Context menu) <br>To link nodes: drag from a node to another to connect nodes. Additionaly if not target selected, It creates a new node <br> Drag and drop: Windows  (1)Ctrl + left click or (2)Mac: ⌘+ left click to move nodes <br>Press Supr / Backspace to delete nodes or lines </p>

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

    addInfo() {
      alert("In progress");
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
.message{
  text-align: left;
}
</style>
