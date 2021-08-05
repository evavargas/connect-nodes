<template>
  <div>
    <h1>Circle in D3</h1>
    <h2>{{ msg }}</h2>
    <div>
      <svg :height="height" :width="width">
        <g transform="translate(50,50)">
          <!-- <circle
            v-for="c in dotsData"
            :key="c.id"
            :r="c.r"
            :cx="c.x"
            :cy="c.y"
            :fill="c.fill"
            :stroke="c.stroke"
          ></circle> -->
        </g>
      </svg>
    </div>
    <div class="canvas" id="canvas"></div>
  </div>
</template>

<script>
import * as d3 from "d3";
export default {
  name: "PackChart",
  props: ["dotsData"],
  data() {
    return {
      msg: "😃",
      height: 600,
      width: 600,
    };
  },
  mounted() {
    this.chart();
  },
  methods: {
    chart() {
      const width = 300;
      const boxHeight = 300;
      const svg = d3
        .select("#canvas")
        .append("svg")
        .attr("viewBox", [0, 0, width, boxHeight]);

      let nodes = [];
      nodes.push([width / 2, boxHeight / 1.5]);
      nodes.push([width / 4, boxHeight / 3]);
      nodes.push([width / 1.5, boxHeight / 3]);

      // Append the nodes to the svg element
      for (let i = 0; i < nodes.length; i++) {
        svg
          .append("circle")
          .attr("cx", nodes[i][0])
          .attr("cy", nodes[i][1])
          .attr("r", 20)
          .style("fill", "green");
      }
      // Create a horizontal link from the first node to the second
      let links = [];
      // Link from the first node to the second
      links.push(
        d3.linkHorizontal()({
          source: nodes[0],
          target: nodes[1],
        })
      );

      // Link from the first node to the third
      links.push(
        d3.linkHorizontal()({
          source: nodes[0],
          target: nodes[2],
        })
      );

      // Append the links to the svg element
      for (let i = 0; i < links.length; i++) {
        svg
          .append("path")
          .attr("d", links[i])
          .attr("stroke", "black")
          .attr("fill", "none");
      }

      return svg.node();
    },
  },
  computed: {},
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.canvas {
  background-color: bisque;
}
</style>
