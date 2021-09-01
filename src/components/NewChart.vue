<template>
  <div ref="resizeRef">
    <svg ref="svgRef" class="svgRef">
      <g class="graph"></g>
    </svg>
  </div>
</template>

<script>
import { onMounted, ref, watchEffect } from "vue";
import * as d3 from "d3";

export default {
  name: "NewChart",
  props: { datashapes: Array, datalinks: Array },
  setup(props) {
    // create ref to pass to D3 for DOM manipulation
    const svgRef = ref(null);
    //width and heigt for rect of nodes
    const width = 60;
    const height = 40;

    onMounted(() => {
      // pass ref with DOM element to D3, when mounted (DOM available)
      const svg = d3.select(svgRef.value);
      //this g is the container of graph
      var g = svg.select(".graph");
      //group of lines
      var allLinks = g.append("g").attr("class", "allLinkGroup");
      //group of nodes
      var allNodes = g.append("g").attr("class", "allNodeGroup");
      //group of text of lines
      var allTexts = g.append("g").attr("class", "allTextGroup");

      //zoom on area graph
      svg.call(
        d3
          .zoom()
          .extent([
            [0, 0],
            [width * 10, height * 10],
          ])
          .scaleExtent([1, 10])
          .on("zoom", zoomed)
      );
      function zoomed() {
        g.attr("transform", d3.event.transform);
      }

      // whenever any dependencies (like data) change, call this!
      watchEffect(() => {
        var link, node, linkText;
        //join links to graph
        link = allLinks
          .selectAll(".line")
          .data(props.datalinks)
          .enter()
          .append("line")
          .classed("line", true);

        //text of links
        linkText = allTexts
          .selectAll("text")
          .data(props.datalinks)
          .enter()
          .append("text")
          .text((d) => d.text)
          .attr("class", "linetext");

        //join nodes to graph
        node = allNodes
          .selectAll(".node")
          .data(props.datashapes)
          .enter()
          .append("g")
          .attr("class", "node");

        //joinning square to nodes
        node
          .append("svg:rect")
          .attr("class", "rect")
          .attr("width", width + "px")
          .attr("height", height + "px");

        //joinning text to node
        node
          .append("text")
          .text((d) => d.text)
          .attr("text-anchor", "start")
          .attr("dominant-baseline", "auto")
          .attr("class", "nodetext")
          .attr("x", 0)
          .attr("y", 8);

        const simulation = d3
          .forceSimulation()
          // add some collision detection so they don't overlap
          .force("collide", d3.forceCollide().radius(30))
          // and draw them around the centre of the space
          .force("center", d3.forceCenter(width * 5, height * 5))
          //selecting nodes for simulation
          .nodes(props.datashapes)
          //selecting links for simulation
          .force(
            "link",
            d3
              .forceLink(props.datalinks)
              .id((d) => d.id)
              .distance(100)
          )
          .on("tick", tick);

        function tick() {
          //where the node line is pinned
          link
            .attr("x1", (d) => d.source.x + width / 2)
            .attr("y1", (d) => d.source.y + height)
            .attr("x2", (d) => d.target.x + width / 2)
            .attr("y2", (d) => d.target.y);
          //text position relative to link
          linkText
            .attr("x", function (d) {
              return (d.source.x + d.target.x) / 2;
            })
            .attr("y", function (d) {
              return (d.source.y + d.target.y) / 2;
            });
          //node position
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
            simulation.alphaTarget(0.8).restart();
          }
        }
        function dragged(d) {
          d.fx = d3.event.x;
          d.fy = d3.event.y;
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
.rect {
  fill: #f7e178;
  stroke: #000000;
}
svg {
  width: 80%;
  height: 960px;
}
.linetext {
  fill: #f43b86;
}
.nodetext {
  fill: #3d087b;
}
.linetext,
.nodetext {
  font-size: 9px;
}
.line {
  fill: #ffffff00;
  stroke: #f43b86;
}
</style>
