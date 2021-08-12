<template>
  <div>
    <Chart  :dotsData="nodes" />
  </div>
</template>

<script>
import * as d3 from "d3";
import axios from 'axios';
import Chart from "./components/Chart.vue";
export default {
  name: "app",
  components: {
    Chart,
  },
  data: function () {
    return {
      entryData: [],
      nodes:[]
    };
  },
  mounted() {
   // this.fetchEntryData();
  this.getNodes()
  },
  methods: {

    async fetchEntryData() {
      let data = await d3.json("./dots.json");
      this.entryData = data;
    },
    async getNodes(){
     await axios.get('./dots.json') 
     .then(response=> {
       let data= response.data;
       console.log(data)
         this.nodes =data;
         console.log(this.nodes)
     })
     .catch((error) => {
       console.log(error);
     });
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
