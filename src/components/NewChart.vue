<template>
  <div class="canvasData" id="canvasData" v-if="datashapes"></div>
</template>

<script>
import * as d3 from "d3";
export default {
  name: "NewChart",
  props: { datashapes: Array, datalinks: Array },
  data() {
    return {
      width: "600",
      height: "600",
      svg: "",
    };
  },
  async created() {},
  mounted() {
    this.initialize();
  },
  updated() {},
  unmounted() {},
  methods: {
    //selecting area
    initialize() {
      // apply fixed positions found in localStorage


      this.width = window.innerWidth * 0.66;
      this.height = window.innerHeight * 0.7;
      this.svg = d3
        .select("#canvasData")
        .append("svg")
        .attr("viewBox", [0, 0, this.width, this.height])
        .attr("transform", "translate(0,00)");
      var g = this.svg.append("g").attr("cursor", "grab");

      var link = g
        .selectAll(".line")
        .data(this.datalinks)
        .join("line")
        .classed("line", true)
        .attr("transform", "translate(150,150)");
      var node = g
        .selectAll(".node")
        .data(this.datashapes)
        .enter()
        .append("g")
        .attr("class", "node")
        .classed("grabbing", (d) => d.fx !== undefined)
        .attr("transform", "translate(150,150)");

      node
        .append("svg:rect")
        .attr("class", "node")
        .classed("rect", true)
        .attr("transform", "translate(150,150)");

      node
        .append("text")
        .text((d) => d.text)
        .attr("text-anchor", "middle")
        .attr("dominant-baseline", "end")
        .attr("font-size", "8px")
        .attr("x", (d) => d.x + 15)
        .attr("y", (d) => d.y + 10)
        .attr("transform", "translate(150,150)");

      //force simulation
      const simulation = d3
        .forceSimulation()
        .nodes(this.datashapes)
        .force(
          "link",
          d3
            .forceLink(this.datalinks)
            .id((d) => d.id)
            .distance(100)
        )
        .on("tick", tick);
      function tick() {
        link
          .attr("x1", (d) => d.source.x)
          .attr("y1", (d) => d.source.y)
          .attr("x2", (d) => d.target.x)
          .attr("y2", (d) => d.target.y);
        node.attr("transform", function (d) {
          return "translate(" + d.x + "," + d.y + ")";
        });
      }
      const drag = d3
        .drag()
        .on("start", dragstarted)
        .on("drag", dragged)
        .on("end", dragended);

      function dragstarted() {
        d3.select(this).raise();
        d3.select(this).classed("grabbing", true);
        simulation.alphaTarget(0.3).restart();
      }
      function dragged() {
        d3.select(this)
          .attr("x", (d) => (d.x += d3.event.dx))
          .attr("y", (d) => (d.y += d3.event.dy));
        simulation.alpha(1).restart();
      }
      function dragended() {
        if (!d3.event.active) simulation.alphaTarget(0);
      }

      node.call(drag).on("click", click);

      function click(d) {
        console.log(d);
        delete d.fx;
        delete d.fy;
        d3.select(this).classed("grabbing", false);
        simulation.alpha(1).restart();
      }

      this.svg.call(
        d3
          .zoom()
          .extent([
            [0, 0],
            [this.width, this.height],
          ])
          .scaleExtent([1, 8])
          .on("zoom", zoomed)
      );
      function zoomed() {
        console.log(g);
        g.attr("transform", d3.event.transform);
      }

      function restart() {
        console.log(this);

        node.enter().append("g").attr("class", "node").merge(node);
        link.enter().append("line").attr("class", "line").merge(link);
        simulation.alpha(1).restart();
      }
      restart();

      return this.svg.node();
    },
  },
};
</script>

<style>
.canvasData {
  background-color: #eeeeee;
}

.rect {
  fill: #ffe35f;
  stroke: #3d087b;
  width: 30px;
  height: 20px;
  color: black;
}
.node .rect {
  fill: #ffe35f;
  stroke: #3d087b;
  width: 30px;
  height: 20px;
  color: black;
}
svg {
  display: inline;
}
.line {
  fill: #ffffff00;
  stroke: #f43b86;
}
.node.grabbing {
  cursor: -webkit-grabbing;
  cursor: -moz-grabbing;
}
.example {
  width: 300px;
  height: 300px;
}
.addButton {
  background-color: rgb(236, 100, 100);
}
</style>
