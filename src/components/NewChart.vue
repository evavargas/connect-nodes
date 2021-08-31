<template>
  <!-- <div class="canvasData" id="canvasData" v-if="datashapes"></div> -->
  <div ref="resizeRef">
    <svg ref="svgRef" class="svgRef">
      <g class="graph" transform="translate(100,100)"></g>
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
  setup(props) {
    // create ref to pass to D3 for DOM manipulation
    const svgRef = ref(null);

    // create another ref to observe resizing, since observing SVGs doesn't work!
    //const { resizeRef, resizeState } = useResizeObserver();

    onMounted(() => {
      // pass ref with DOM element to D3, when mounted (DOM available)
      const svg = d3.select(svgRef.value);
      //this g is the container of graph
      var g = svg.select(".graph");
      //group of lines
      var allLinks = g.append("g").attr("class", "allLinkGroup");

      var allNodes = g.append("g").attr("class", "allNodeGroup");

      var allTexts = g.append("g").attr("class", "allTextGroup");

      //zoom on area graph
      svg.call(
        d3
          .zoom()
          .extent([
            [0, 0],
            [600, 600],
          ])
          .scaleExtent([1, 10])
          .on("zoom", zoomed)
      );
      function zoomed() {
        console.log(g);
        g.attr("transform", d3.event.transform);
      }

      // whenever any dependencies (like data, resizeState) change, call this!
      watchEffect(() => {
        //const { width, height } = resizeState.dimensions;
        var link, node, linkText;
        //center graph
        
        //links update
        link = allLinks
          .selectAll(".line")
          .data(props.datalinks)
          .enter()
          .append("line")
          .classed("line", true)

        //text of links
        linkText = allTexts
        .selectAll('text')
        .data(props.datalinks)
        .enter()
        .append("text")
        .text((d)=> d.text)
        .attr("class", "linetext")
        //nodes updates
        node = allNodes
          .selectAll(".node")
          .data(props.datashapes)
          .enter()
          .append("g")
          .attr("class", "node")
        //joinning square to nodes
        node
          .append("svg:rect")
          .attr("class", "rect")

        //joinning text to node
        node
          .append("text")
          .text((d) => d.text)
          .attr("text-anchor", "start")
          .attr("dominant-baseline", "auto")
          .attr("class", "nodetext")
          .attr("x",0)
          .attr("y",6)

        const simulation = d3
          .forceSimulation()
          .nodes(props.datashapes)
          //.force("center", d3.forceCenter(600 / 2, 600 / 2))
          .force(
            "link",
            d3
              .forceLink(props.datalinks)
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
          linkText
        .attr('x', function (d) { return (d.source.x + d.target.x) / 2 })
        .attr('y', function (d) { return (d.source.y + d.target.y) / 2 })
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
        node.call(drag);

        function dragstarted() {
         if (!d3.event.active) {
        simulation.alphaTarget(0.8).restart()
      }
    }
        function dragged (d) {
      d.fx = d3.event.x
      d.fy = d3.event.y
    }
        function dragended() {
          if (!d3.event.active) simulation.alphaTarget(0);
        }
      });
    });
    return { svgRef };
  },
};
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
}
svg {
  width: 600px;
  height: 600px;
}
.linetext{
  color: #f43b86;
}
.nodetext{
  color: #3d087b;
}
.linetext, .nodetext{
  font-size: 8px;
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
