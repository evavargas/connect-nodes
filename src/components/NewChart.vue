<template>
  <div>
    <div v-if="shapes && links" class="canvasData" id="canvasData">
      <button @click="createNew">New</button>
    </div>
  </div>
</template>

<script>
import * as d3 from "d3";
export default {
  name: "NewChart",
  props: ["shapes", "links"],
  data() {
    return {
      width: "600",
      height: "600",
      svg: "",
      link:'',
      node:'',
      simulation: "",
      drag:'',
    };
  },
  async created() {
    
  },
mounted() {
    
  },
  updated() {
    this.initialize();
  },
  unmounted() {},
  methods: {
    //selecting area
    initialize() {
      this.svg = d3
        .select("#canvasData")
        .append("svg")
        .attr("viewBox", [0, 0, this.width, this.height])
        .attr("transform", "translate(0,0)");

      this.link = this.svg
        .selectAll(".link")
        .data(this.links)
        .join("line")
        .classed("link", true)
        .attr("transform", "translate(45,35)");

      this.node = this.svg
        .selectAll(".rect")
        .data(this.shapes)
        .join("rect")
        .classed("rect", true)
        .classed("fixed", (d) => d.fx !== undefined)
        .attr("transform", "translate(45,35)");
      
      //force simulation
      this.simulation = d3
        .forceSimulation()
        .nodes(this.shapes)
        .force("charge", d3.forceManyBody().strength())
        .force("center", d3.forceCenter(this.width / 3, this.height / 3))
        .force("link", d3.forceLink().links(this.links))
        .on("tick", this.tick());
      
      this.drag = d3
        .drag()
        .on("start", this.dragstart)
        .on("drag", this.dragged);

      this.node.call(this.drag).on("click", this.click);
      return this.svg.node();
    },
    tick() {
      this.link
      .attr("x1", d => d.source.x)
      .attr("y1", d => d.source.y)
      .attr("x2", d => d.target.x)
      .attr("y2", d => d.target.y);
      this.node
      .attr("x", d => d.x)
      .attr("y", d => d.y);
    },
    click(event, d){
      delete d.fx;
      delete d.fy;
      d3.select(this).classed("fixed", false);
      this.simulation.alpha(1).restart();
    },
    dragstart(){
      d3.select(this).classed("fixed", true);
    },
    dragged(event, d){
      d.fx = this.clamp(event.x, 0, this.width);
      d.fy = this.clamp(event.y, 0, this.height);
      this.simulation.alpha(1).restart();
    },
    clamp(x, lo, hi) {
      return x < lo ? lo : x > hi ? hi : x;
      },
    createNew() {
      this.svg
        .append("rect")
        .attr("x", (d, i, nodes) => {
          return +nodes[i].previousElementSibling.getAttribute("x") + 60;
        })
        .attr("y", (d, i, nodes) => {
          return +nodes[i].previousElementSibling.getAttribute("y") ;
        })
        .classed("rect", true);
    },

  },
  computed: {},
};
</script>

<style>
.canvasData {
  background-color: #eeeeee;
}
.canvasData :nth-child(2){
  display: none;
}
.rect {
  fill: #ffe35f;
  stroke: #3d087b;
  width: 16px;
  height: 10px;
  color: black;
}
svg {
  display: inline;
}
.line {
  fill: #ffffff00;
  stroke: #f43b86;
}
.fixed{
  fill: #f00;
}
.example {
  width: 300px;
  height: 300px;
}
</style>
