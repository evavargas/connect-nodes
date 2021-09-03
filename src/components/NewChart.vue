<template>
  <div ref="resizeRef" class="resizeRef">
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
    const refShape = ref(props.datashapes);
    //const refLink = ref(props.datalinks)
    //width and heigt for rect of nodes
    const width = 60;
    const height = 40;

    
    onMounted(() => {
      // pass ref with DOM element to D3, when mounted (DOM available)
      const svg = d3.select(svgRef.value);
      //this g is the container of graph
      var g = svg.select(".graph");
      //svg marker for arrow
      var defsarrow = g.append("svg:defs");
      //group of lines
      var allLinks = g.append("g").attr("class", "allLinkGroup");
      //group of nodes
      var allNodes = g.append("g").attr("class", "allNodeGroup");
      //group of text of lines
      var allTexts = g.append("g").attr("class", "allTextGroup");

      //to arrow
      defsarrow
        .selectAll("marker")
        .data(["end"])
        .enter()
        .append("svg:marker")
        .attr("id", function () {
          return "marker_" + "arrow";
        })
        .attr("markerHeight", 15)
        .attr("markerWidth", 15)
        .attr("markerUnits", "strokeWidth")
        .attr("orient", "auto")
        .attr("class", "arrow")
        .attr("refX", 0)
        .attr("refY", 0)
        .attr("viewBox", "-5 -5 10 10")
        .append("svg:path")
        .attr("d", "M 0,0 m -5,-5 L 5,0 L -5,5 Z");

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
          //.append("line") for tick
          .append("path") //for ticked
          .classed("line", true)
          //adding arrow
          .attr("marker-end", function () {
            return "url(#marker_" + "arrow" + ")";
          });

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
          //.force("center", d3.forceCenter(width * 5, height * 5))
          //selecting nodes for simulation
          .nodes(props.datashapes)
          //selecting links for simulation
          .force(
            "link",
            d3
              .forceLink(props.datalinks)
              .id((d) => d.id)
              .distance(120)
          )
          .on("tick", ticked);
        function ticked() {
          //curve the link
          link.attr("d", positionLink);

          //adding text to links
          linkText
            .attr("x", function (d) {
              return (d.source.x + d.target.x) / 2;
            })
            .attr("y", function (d) {
              return (d.source.y + d.target.y) / 2;
            });
          //position of nodes linked
          node.attr("transform", positionNode);
        }
        function positionLink(d) {
          var offset = 30;

          var midpoint_x = (d.source.x + d.target.x) / 2;
          var midpoint_y = (d.source.y + d.target.y) / 2;

          var dx = d.target.x - d.source.x;
          var dy = d.target.y - d.source.y;

          var normalise = Math.sqrt(dx * dx + dy * dy);

          var offSetX = midpoint_x + offset * (dy / normalise);
          var offSetY = midpoint_y - offset * (dx / normalise);

          return (
            "M" +
            (d.source.x + width / 2) +
            "," +
            (d.source.y + height) +
            "S" +
            offSetX +
            "," +
            offSetY +
            " " +
            (d.target.x + width / 2) +
            "," +
            d.target.y
          );
        }
        // move the node based on forces calculations
        function positionNode(d) {
          // keep the node within the boundaries of the svg
          if (d.x < 0) {
            d.x = 0;
          }
          if (d.y < 0) {
            d.y = 0;
          }
          if (d.x > 1000) {
            d.x = 1000;
          }
          if (d.y > 1000) {
            d.y = 1000;
          }
          return "translate(" + d.x + "," + d.y + ")";
        }
        //addNode
        
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
            simulation.alphaTarget(0.3).restart();
          }
        }
        function dragged(d) {
          d.fx = d3.event.x;
          d.fy = d3.event.y;
        }
        function dragended() {
          if (!d3.event.active) simulation.alphaTarget(0);
        }

        function addNode(x){
          console.log("click")
          refShape.value.push(x)
        }
        svg.on("click", function(){
          if (!d3.event.active)
          addNode({
        id: refShape.value.length+1,
        y: 0,
        x: 0,
        text: "example",
        index: null,
      })
        }
        )
      });
    });
    return { svgRef };
  },
};
</script>

<style>
.svgRef{
  background-color: #F7F6F2;
}
svg {
  width: 80%;
  height: 960px;
}
svg:hover{
  cursor: crosshair;
}
.node:hover{
  cursor: grab;
}
.graph{
cursor: move;
}
.rect {
  fill: #f7e178;
  stroke: black;
}
.rect:active {
    cursor: grabbing;
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
.arrow {
  fill: aquamarine;
  stroke: blue;
}
</style>
