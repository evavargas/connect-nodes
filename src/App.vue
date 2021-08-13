<template>
  <div>
    <h1>Nodes linked with D3</h1>
    <h2>{{ msg }}</h2>
    <form class="form-addnode">
      <h1>To Add a new Node</h1>
      <label>Node name: </label>
      <input type="text" v-model="node.id" />
      <label>Select parent: </label>
      <select v-model="nodeparentselected">
        <option
          v-for="(node, index) in nodes"
          :key="node.id + index"
          :value="[node.position[0],node.position[1]]"
        >
          {{ node.id }}, {{ node.position }}
        </option>
        <option :value="[nodeposition.x,nodeposition.y]">None</option>
      </select>
      <label>Select position in X: </label>
      <input type="number" min="0" max="100" v-model="nodeposition.x" />
      <label>Select position in Y: </label>
      <input type="number" min="0" max="100" v-model="nodeposition.y" />
      <label></label>
      <div @click="save" class="btn-addnode">Add Node</div>
    </form>
    <div>
      <Chart :dotsData="nodes" />
    </div>
  </div>
</template>

<script>
import axios from "axios";
import Chart from "./components/Chart.vue";
export default {
  name: "app",
  components: {
    Chart,
  },
  data: function () {
    return {
      msg: "💃🕺",
      nodes: [],
      node: {
        id: "",
        position: [],
        parentPosition: {
          x:0,
          y:0
          },
      },
      nodeparentselected:'',
      nodeposition: {
        x: 0,
        y: 0,
      },
    };
  },
  mounted() {
    this.getNodes();
  },
  methods: {
    async getNodes() {
      await axios
        .get("./dots.json")
        .then((response) => {
          let data = response.data;
          console.log(data);
          this.nodes = data;
          console.log(this.nodes);
        })
        .catch((error) => {
          console.log(error);
        });
    },
    async newNode() {
      let dataSource = this.getData();
      console.log(dataSource)
      //to use axios post pethod:
      // await axios
      // .post(`/dots.json`, dataSource)
      // .then(() => {
      // this.getNodes()
      // })
      // .catch((error) => {
      //   console.log(error);
      // });      
    },
    getData() {
      this.node.parentPosition.x = this.nodeparentselected[0];
      this.node.parentPosition.y = this.nodeparentselected[1];
      return {
        id: this.node.id,
        position: [this.nodeposition.x, this.nodeposition.y],
        parentPosition: [this.node.parentPosition.x,this.node.parentPosition.y]
      };
    },
    async save(){
      this.newNode()
    }
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
