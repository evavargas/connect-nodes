<template>
  <!-- <div class="canvasData" id="canvasData" v-if="datashapes"></div> -->
  <div ref="resizeRef">
    <svg ref="svgRef">
      <g class="graph">
      </g>
    </svg>
  </div>
</template>

<script>
import { onMounted, ref, watchEffect } from "vue";
import * as d3 from "d3";
//import useResizeObserver from "@/use/resizeObserver";

export default {
  name: "NewChart",
  props: { datashapes: Array, datalinks: Array },
  setup(props){
    // create ref to pass to D3 for DOM manipulation
    const svgRef = ref(null);

    // create another ref to observe resizing, since observing SVGs doesn't work!
    //const { resizeRef, resizeState } = useResizeObserver();

    onMounted(()=>{
      // pass ref with DOM element to D3, when mounted (DOM available)
      const svg = d3.select(svgRef.value);
      //this g is the container of graph
        var g = svg.select(".graph")
        
      //group of lines
        var allLinks = g.append("g").attr("class","allLinkGroup");

        var allNodes = g.append("g").attr("class","allNodeGroup");
        

      // whenever any dependencies (like data, resizeState) change, call this!
      watchEffect(()=>{
        //const { width, height } = resizeState.dimensions;
        var link, node;

        //links update
        link = allLinks
          .selectAll(".line")
          .data(props.datalinks)
          .join("line")
          .classed("line", true)
          .attr("transform", "translate(150,150)");

        //nodes updates
        node = allNodes
          .selectAll(".node")
          .data(props.datashapes)
          .enter()
          .append("g").attr("class", "node")
          .attr("transform", "translate(150,150)");
        //joinning square to nodes
        node
          .append("svg:rect")
          .attr("class", "rect")
          .attr("transform", "translate(150,150)");

        //joinning text to node
        node
          .append("text")
          .text((d) => d.text)
          .attr("text-anchor", "middle")
          .attr("dominant-baseline", "end")
          .attr("font-size", "8px")
          .attr("x", (d) => d.x + 15)
          .attr("y", (d) => d.y + 10)
          .attr("transform", "translate(150,150)");
        

        const simulation = d3.forceSimulation()
        .nodes(props.datashapes)
        .force("link", d3.forceLink(props.datalinks).id((d)=> d.id).distance(100))
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
      //mouse events
      const drag = d3
        .drag()
        .on("start", dragstarted)
        .on("drag", dragged)
        .on("end", dragended);

      // mouse events on nodes
      node.call(drag).on("click", click);

      function dragstarted() {
        d3.select(this).raise();
        simulation.alphaTarget(0.3).restart();
      }
      function dragged() {
        d3.select(this)
          .attr("x", (d) => (d.x += d3.event.dx))
          .attr("y", (d) => (d.y += d3.event.dy));
        simulation.alphaTarget(0.3).restart();
      }
      function dragended() {
        if (!d3.event.active) simulation.alphaTarget(0);
      }

      //simulation for mouse events
      function click(d) {
        console.log(d);
        delete d.fx;
        delete d.fy;
        simulation.alpha(1).restart();
      }
      //zoom on area graph
      svg.call(
        d3
          .zoom()
          .extent([
            [0, 0],
            [600, 600],
          ])
          .scaleExtent([1, 8])
          .on("zoom", zoomed)
      );
      function zoomed() {
        console.log(g);
        g.attr("transform", d3.event.transform);
      }
      })
    })
     return { svgRef};
  },
}
</script>

<style>
.canvasData {
  background-color: #eeeeee;
}
.div {
  background-color: rgb(255, 255, 255);
  width: 60px;
  height: 20px;
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
  width: 600px;
  height: 600px;
}
.line {
  fill: #ffffff00;
  stroke: #f43b86;
}

.example {
  width: 300px;
  height: 300px;
}
.addButton {
  background-color: rgb(236, 100, 100);
}
</style>
