<template>
  <link
    rel="stylesheet"
    href="http://code.ionicframework.com/ionicons/2.0.1/css/ionicons.min.css"
  />
  <div>
    <header>
      <div @click="show = !show">
        <i class="ion-navicon"></i>
      </div>
    </header>
    <aside class="box-app" v-if="show" transition="slide">
      <ul>
        <li>
          <div class="button">
            <label for="newChart" class="input">New chart</label>
            <input type="button" id="newChart" value="New Chart" @change="newFile" style="display: none"/>
          </div>
        </li>
        <li>
          <div class="button">
            <label for="files" class="input">Open chart</label>
            <input type="file" id="files" @change="loadTextFromFile" style="display: none" />
          </div>
        </li>
        <li>
          <div class="button">
            <label for="download" class="input">Download chart</label>
            <input type="button" id="download" value="Download" v-on:click="downloadFile()" style="display: none"/>
          </div>
        </li>
      </ul>
    </aside>
    <keep-alive>
      <Chart :datashapes="datanodes" :datalinks="datalinks" />
    </keep-alive>
  </div>
</template>

<script>
import { defineAsyncComponent } from "vue";
import axios from "axios";
import { saveAs } from "file-saver";

export default {
  name: "app",
  components: {
    Chart: defineAsyncComponent(() => import("./components/Chart.vue")),
  },
  data: function () {
    return {
      show: false,
      datanodes: [],
      datalinks: [],
      json: "",
      data: {},
      // nodes: [{ id: 0, x: 0, y: 0, text: "" }],
      // links: [{ id: 0, source: 0, target: 0, text: "" }],
    };
  },
  created() {
    this.getShapes("./chartExample.json");
  },

  methods: {
    loadTextFromFile(ev) {
      const file = ev.target.files[0];
      const reader = new FileReader();

      reader.onload = (e) => {
        {
          this.$emit("load", e.target.result);
          this.json = JSON.parse(e.target.result);
          this.updateChart(this.json);
        }
      };

      reader.readAsText(file);
    },
    updateChart(data) {
      this.datanodes = data.nodes;
      this.datalinks = data.links;
      console.log(this.datanodes);
      console.log(this.datalinks);
    },
    newFile() {
      this.datalinks = [];
      this.datanodes = [];
    },

    // uploadFile() {
    //   this.newFile = this.$refs.file.files[0];
    // },
    // submitFile() {
    //   const fr = new FileReader();
    //   fr.addEventListener("load", e => {
    //     console.log(e.target.result, JSON.parse(fr.result))
    //   })
    //fr.readAsText(this.newFile)
    //this.getShapes(ruta);
    //},
    async getShapes(ruta) {
      let response = await axios.get(ruta);
      this.datanodes = response.data.nodes;
      this.datalinks = response.data.links;
    },
    openFile() {},
    downloadFile() {
      console.log(this.$root.$data.datanodes);
      let listNodes = this.$root.$data.datanodes;
      let listLinks = this.$root.$data.datalinks;
      this.data.nodes = listNodes;
      this.data.links = listLinks;
      console.log(this.data);
      var file = new Blob([JSON.stringify(this.data)], {
        type: "application/json",
      });
      saveAs(file, "chart.json");
      //var dataString = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(this.data));

      // async getInfo() {
      //   let response = await axios.get("./Info.json");
      //   this.nodes = response.data;
      // },

      // addNode() {
      //   this.datashapes = [...this.datashapes, this.newNode];
      // },
    },
  },
};
</script>

<style >
header {
  width: 100%;
  height: 56px;
  position: fixed;
  top: 0;
  left: 0;
  z-index: 80;
  background-color: #fff;
  box-shadow: 2px 2px 5px 0 rgba(0, 0, 0, 0.095);
  padding: 8px 16px;
  overflow: hidden;
  box-sizing: border-box;
}
.ion-navicon {
  font-size: 24px;
}
.box-app {
  background-color: #fff;
  width: 250px;
  position: fixed;
  top: 55px;
  left: 0;
  bottom: 0;
  z-index: 1;
  display: flex;
  justify-content: left;
  align-items: left;
  box-shadow: 0 3px 6px rgba(0, 0, 0, 0.16), 0 3px 6px rgba(0, 0, 0, 0.23);
}
aside h2 {
  font-weight: 300;
  color: #afafaf;
}
.box-app .button {
  background-color: transparent;
  border: 0;
  width: 80px;
  height: 40px;
  outline: none;
  cursor: pointer;
  border-radius: 50%;
  transition: all 0.3s ease-in-out;
}
.button input, .button .input{
  width: 100px;
}
.button:active {
  background-color: rgba(0, 0, 0, 0.1);
}
.slide-transition {
  transition: all 0.3s cubic-bezier(0.65, 0.05, 0.36, 1);
}
.slide-enter,
.slide-leave {
  left: -100%;
}
</style>
