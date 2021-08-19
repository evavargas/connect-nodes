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
    };
  },
  async created() {},
  mounted() {},
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
        .attr("transform", "translate(45,35)");

      const link = this.svg
          .selectAll(".link")
          .data(this.links)
          .join("line")
          .classed("link", true)
          .attr("transform", "translate(45,35)"),
        node = this.svg
          .selectAll(".rect")
          .data(this.shapes)
          .join("rect")
          .classed("rect", true)
          .classed("fixed", (d) => d.fx !== undefined)
          .attr("transform", "translate(45,35)");

      //force simulation
      const simulation = d3
        .forceSimulation()
        .nodes(this.shapes)
        .force("link",d3.forceLink(this.links).id((d) => d.id).distance(function(){
          return 110;
        }))
        .on("tick", tick);

      const drag = d3.drag().on("start", dragstart).on("drag", dragged);

      node.call(drag).on("click", click);

      function tick() {
        link
          .attr("x1", (d) => d.source.x)
          .attr("y1", (d) => d.source.y)
          .attr("x2", (d) => d.target.x)
          .attr("y2", (d) => d.target.y);
        node.attr("x", (d) => d.x).attr("y", (d) => d.y);
      }
      function click(d) {
        console.log(d);
        delete d.fx;
        delete d.fy;
        d3.select(this).classed("fixed", false);
        simulation.alpha(1).restart();
      }
      function dragstart() {
        d3.select(this).classed("fixed", true);
      }
      function dragged() {
        d3.select(this)
          .attr("x", (d) => (d.x += d3.event.dx))
          .attr("y", (d) => (d.y += d3.event.dy));
        simulation.alpha(1).restart();
      }

      return this.svg.node();
    },
    createNew() {
      this.svg
        .append("rect")
        .attr("x", (d, i, nodes) => {
          return +nodes[i].previousElementSibling.getAttribute("x") + 60;
        })
        .attr("y", (d, i, nodes) => {
          return +nodes[i].previousElementSibling.getAttribute("y");
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
  width: 60px;
  height: 40px;
  color: black;
  cursor: move;
}
svg {
  display: inline;
}
.line {
  fill: #ffffff00;
  stroke: #f43b86;
}
.fixed {
  fill: #f00;
}
.example {
  width: 300px;
  height: 300px;
}
</style>
