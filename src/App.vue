<template>
  <div>
    <nav class="menu">
      <div>
        <ul>
          <li>
            <input type="file" v-on:change="selectFile()" ref="file" />
            <button v-on:click="openFile()">Open File</button>
          </li>
          <li>
            <button v-on:click="downloadFile()">Download</button>
          </li>
        </ul>
      </div>
    </nav>
    <h2>{{ msg }}</h2>

    <keep-alive>
      <Chart :datashapes="datanodes" :datalinks="datalinks" />
    </keep-alive>
    <p class="message">
      To add a node: (1) right click. (Context menu) <br />To link nodes: drag
      from a node to another to connect nodes. Additionaly if not target
      selected, It creates a new node <br />
      Drag and drop: Windows (1)Ctrl + left click or (2)Mac: ⌘+ left click to
      move nodes <br />Press Supr / Backspace to delete nodes or lines
    </p>
  </div>
</template>

<script>
import { defineAsyncComponent } from "vue";
import axios from "axios";
import { saveAs } from 'file-saver';

export default {
  name: "app",
  components: {
    Chart: defineAsyncComponent(() => import("./components/NewChart.vue")),
  },
  data: function () {
    return {
      msg: "💃🕺",
      datanodes: [],
      datalinks: [],
      newFile: null,
      data:{}
      // nodes: [{ id: 0, x: 0, y: 0, text: "" }],
      // links: [{ id: 0, source: 0, target: 0, text: "" }],
    };
  },
  created() {
    this.loadFile("./chartExample.json");
  },

  methods: {
    loadFile(ruta) {
      this.getShapes(ruta);
    },
    async getShapes(ruta) {
      let response = await axios.get(ruta);
      this.datanodes = response.data.nodes;
      this.datalinks = response.data.links;
    },
    downloadFile() {
      console.log(this.$root.$data.datanodes);
      let listNodes = this.$root.$data.datanodes;
      let listLinks = this.$root.$data.datalinks;
      this.data.nodes = listNodes;
      this.data.links = listLinks;
      console.log(this.data);
      var file = new Blob([JSON.stringify(this.data)], {type:'application/json'});
      saveAs(file, "chart.json")
      //var dataString = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(this.data));

    },


    // async getInfo() {
    //   let response = await axios.get("./Info.json");
    //   this.nodes = response.data;
    // },

    // addNode() {
    //   this.datashapes = [...this.datashapes, this.newNode];
    // },
    selectFile() {
      this.newFile = this.$refs.file.files[0];
    },
    openFile() {},
  },
};
</script>

<style>
.menu {
  background-color: #fcf0f0;
}
.message {
  text-align: left;
}
</style>
