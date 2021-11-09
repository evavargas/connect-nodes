<template>
  <div>
    <header>
      <div class="menu-header">
        <div @click="showOptions = !showOptions">
          <MenuOutline class="ion-icon" v-if="!showOptions" />
          <MdCloseCircleOutlineIcon class="ion-icon" v-if="showOptions" />
        </div>
        <div @click="showHelp = !showHelp">
          <MdHelpCircleIcon class="ion-icon" v-if="!showHelp" />
          <MdCloseCircleOutlineIcon class="ion-icon" v-if="showHelp" />
        </div>
      </div>
    </header>
    <aside class="box-opt" v-if="showOptions">
      <ul>
        <li>
          <div class="button">
            <label for="newChart" class="input"
              ><MdDocumentIcon class="ion-icon" />New chart</label
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
              ><MdFolderOpenIcon class="ion-icon" />Open chart</label
            >
            <input
              type="file"
              id="files"
              accept=".json"
              @change="openFile"
              style="display: none"
            />
          </div>
        </li>
        <li>
          <div class="button">
            <MdDownloadIcon class="ion-icon" />
            <a
              class="input"
              id="downloadFile"
              :href="urlFile"
              download="My_chart"
              v-on:click="downloadChart()"
              >Download Chart</a
            >
          </div>
        </li>
      </ul>
    </aside>
    <aside class="box-help" v-if="showHelp" transition="slide">
      <ul>
        Add a node
        <li>Right click. (Context menu) or button <strong>Add Node</strong></li>
      </ul>
      <ul>
        Link nodes
        <li>
          Drag from a node to another to connect nodes. Additionaly if not
          target is selected, It creates a new node
        </li>
      </ul>
      <ul>
        Drag and drop
        <li>Windows: Ctrl + left click.</li>
        <li>Mac: ⌘ + left click.</li>
      </ul>
      <ul>
        Zoom
        <li>Scrool with mouse</li>
      </ul>
      <ul>
        Remove nodes or links
        <li>
          Select the element and Press Supr or Backspace to delete nodes or
          lines
        </li>
        <li>Also, use the button <strong>Delete element</strong></li>
      </ul>
    </aside>
    <div class="body-chart">
      <keep-alive>
        <Chart
          :datashapes="datanodes"
          :datalinks="datalinks"
          :key="componentKey"
        />
      </keep-alive>
    </div>
  </div>
</template>

<script>
//import { defineAsyncComponent } from "vue";
import MenuOutline from "vue-ionicons/dist/ios-menu.vue"; //Menu
import MdHelpCircleIcon from "vue-ionicons/dist/md-help-circle.vue"; //Help
import MdCloseCircleOutlineIcon from "vue-ionicons/dist/md-close-circle-outline.vue"; //Close
import MdDocumentIcon from "vue-ionicons/dist/md-document.vue"; //New
import MdFolderOpenIcon from "vue-ionicons/dist/md-folder-open.vue"; //Open
import MdDownloadIcon from "vue-ionicons/dist/md-download.vue"; //Download
//Component
import Chart from "./components/Chart.vue";

export default {
  name: "app",
  components: {
    Chart, //: defineAsyncComponent(() => import("./components/Chart.vue")),
    MenuOutline,
    MdDocumentIcon,
    MdFolderOpenIcon,
    MdDownloadIcon,
    MdHelpCircleIcon,
    MdCloseCircleOutlineIcon,
  },
  data: function () {
    return {
      showHelp: false,
      showOptions: false,
      datanodes: [],
      datalinks: [],
      json: "",
      data: {},
      urlFile: "",
      componentKey: 0,
    };
  },
  methods: {
    openFile(ev) {
      const file = ev.target.files[0];
      const reader = new FileReader();
      if (file) {
        reader.addEventListener("error", () => {
          alert(`Error occurred reading file: ${file.name}`);
        });
      }
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
        //n.index = null;
      });
      this.componentKey += 1;
      this.datalinks = [];
      this.datanodes = [];
      this.data = [];
      this.datanodes = incomingNodes;
      this.datalinks = incomingLinks;
    },
    newFile() {
      this.componentKey += 1;
      this.datalinks = [];
      this.datanodes = [];
      this.data = [];
    },

    downloadChart() {
      let listNodes = this.$root.$data.datanodes;
      let listLinks = this.$root.$data.datalinks;
      let data = {};
      data.nodes = listNodes;
      data.links = listLinks;
      var file = new Blob([JSON.stringify(data)], {
        type: "application/json",
      });
      let url = window.URL.createObjectURL(file);
      this.urlFile = url;
    },
  },
};
</script>

<style >
body{
  margin: 0;
}
.menu-header {
  display: flex;
  justify-content: space-between;
}
header {
  width: 100%;
  height: 36px;
  position: fixed;
  top: 0;
  left: 0;
  z-index: 1;
  background-color: #cdefff;
  box-shadow: 2px 2px 5px 0 rgba(0, 0, 0, 0.095);
  padding: 8px 16px;
  overflow: hidden;
  box-sizing: border-box;
}
.ion-icon {
  font-size: 24px;
  display: inline;
  stroke: rgb(78, 102, 238);
  fill: rgb(37, 66, 232);
  stroke-width: 8px;
  stroke-opacity: 0.9;
}
.ion-icon:active{
  fill: rgba(37, 66, 232, 0.349);
}

.box-opt {
  background-color: #cdefff;
  width: 180px;
  position: fixed;
  top: 37px;
  left: 0;
  bottom: 0;
  z-index: 2;
  display: flex;
  justify-content: left;
  box-shadow: 0 3px 6px rgba(0, 0, 0, 0.16), 0 3px 6px rgba(0, 0, 0, 0.23);
}
.box-opt ul {
  list-style: none;
  padding: 0;
}
.box-opt ul li{
  padding: 0.2rem 0.6rem;
}

.box-opt .button {
  background-color: transparent;
  border: 0;
  width: 140px;
  margin: 2px 0px;
  padding: 0.7rem;
  outline: none;
  cursor: pointer;
  border-radius: 20%;
  transition: all 0.3s ease-in-out;
  font-size: 16px;
  box-sizing: border-box;
  position: relative;
}
.button .input {
  width: 140px;
  padding: 1rem 0;
  text-decoration: none;
  color: black;
}
.input:hover {
  text-shadow: 1px 1px rgba(37, 66, 232, 0.349);
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
.box-help {
  background-color: #cdefff;
  width: 180px;
  position: fixed;
  top: 37px;
  right: 0;
  bottom: 0;
  z-index: 2;
  box-shadow: 0 3px 6px rgba(0, 0, 0, 0.16), 0 3px 6px rgba(0, 0, 0, 0.23);
}
 @media only screen and (min-width: 1500px) {
  .body-chart {
    display: flex;
    justify-content: center;
  }
}
/*
@media only screen and (min-width: 1280px) and (max-width: 1350px) {
  .body-chart {
    display: flex;
    justify-content: left;
  }
} */
</style>
