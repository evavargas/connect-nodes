<template>
  <div>
    <header>
      <div @click="show = !show">
        <MenuOutline class="ion-icon" />
      </div>
    </header>
    <aside class="box-app" v-if="show" transition="slide">
      <ul>
        <li>
          <div class="button">
            <label for="newChart" class="input"
              ><DocumentOutline class="ion-icon" />New chart</label
            >
            <input
              type="button"
              id="newChart"
              value="New Chart"
              v-on:click="newFile()"
              style="display: none"
            />
          </div>
        </li>
        <li>
          <div class="button">
            <label for="files" class="input"
              ><FolderOpenOutline class="ion-icon" />Open chart</label
            >
            <input
              type="file"
              id="files"
              @change="openFile"
              style="display: none"
            />
          </div>
        </li>
        <li>
          <div class="button">
            <label for="download" class="input"
              ><DownloadOutline class="ion-icon" />Download chart</label
            >
            <input
              type="button"
              id="download"
              value="Download"
              v-on:click="downloadFile()"
              style="display: none"
            />
          </div>
        </li>
      </ul>
    </aside>
    <keep-alive>
      <Chart
        :datashapes="datanodes"
        :datalinks="datalinks"
        :key="componentKey"
      />
    </keep-alive>
  </div>
</template>

<script>
//import { defineAsyncComponent } from "vue";
//import axios from "axios";
import { saveAs } from "file-saver";
import MenuOutline from "vue-ionicons/dist/ios-menu.vue";
import FolderOpenOutline from "vue-ionicons/dist/ios-folder-open.vue";
import DocumentOutline from "vue-ionicons/dist/ios-document.vue";
import DownloadOutline from "vue-ionicons/dist/ios-download.vue";
import Chart from "./components/Chart.vue";
export default {
  name: "app",
  components: {
    Chart, //: defineAsyncComponent(() => import("./components/Chart.vue")),
    MenuOutline,
    FolderOpenOutline,
    DocumentOutline,
    DownloadOutline,
  },
  data: function () {
    return {
      show: false,
      datanodes: [],
      datalinks: [],
      json: "",
      data: {},
      componentKey: 0,
    };
  },
  beforeUpdate() {
    this.verifyrenderOk();
  },
  beforeMount() {
    this.verifyrenderOk();
  },
  methods: {
    verifyrenderOk() {
      //prevent re-render
      var buttons = document.getElementsByClassName("btn-inner");
      if (buttons.length > 2) {
        buttons[0].remove();
        buttons[1].remove();
      }
    },
    openFile(ev) {
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
      this.componentKey += 1;
      let incomingNodes = data.nodes;
      let incomingLinks = data.links;

      incomingLinks.forEach((n) => {
        n.source = n.source.id;
        n.target = n.target.id;
        n.index = null;
      });
      this.datanodes = incomingNodes;
      this.datalinks = incomingLinks;
    },
    newFile() {
      this.componentKey += 1;
      this.datalinks = [];
      this.datanodes = [];
      this.data = [];
    },

    downloadFile() {
      let listNodes = this.$root.$data.datanodes;
      let listLinks = this.$root.$data.datalinks;
      let data = {};
      data.nodes = listNodes;
      data.links = listLinks;
      var file = new Blob([JSON.stringify(data)], {
        type: "application/json",
      });
      saveAs(file, "chart.json");
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
.ion-icon {
  font-size: 24px;
  display: inline;
  stroke: rgb(8, 63, 72);
  fill: rgb(219, 250, 255);
  stroke-width: 8px;
  stroke-opacity: 0.9;
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
.box-app ul {
  list-style: none;
}
aside h2 {
  font-weight: 300;
  color: #afafaf;
}
.box-app .button {
  background-color: transparent;
  border: 0;
  width: 136px;
  margin: 15px 0px;
  padding: 0.7rem;
  outline: none;
  cursor: pointer;
  border-radius: 20%;
  transition: all 0.3s ease-in-out;
  font-size: 16px;
  box-sizing: border-box;
}
.button .input {
  width: 136px;
}
.input:hover {
  text-shadow: 1px 1px rgb(219, 250, 255);
}
.button .ion-icon {
  font-size: 14px;
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
