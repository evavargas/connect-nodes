<template>
  <div ref="resizeRef" class="resizeRef">
    <svg ref="svgRef" class="svgRef">
      <g class="graph"></g>
    </svg>
  </div>
</template>

<script>
import { onMounted, ref } from "vue";
import * as d3 from "d3";

export default {
  name: "NewChart",
  props: { datashapes: Array, datalinks: Array },
  setup(props) {
    // create ref to pass to D3 for DOM manipulation
    const svgRef = ref(null);
    var refShape = ref(props.datashapes);
    var refLink = ref(props.datalinks);
    //width and heigt for rect of nodes
    const width = 60;
    const height = 40;
    var margin = { top: 20, right: 90, bottom: 30, left: 90 },
      widthGraph = 960 - margin.left - margin.right,
      heightGraph = 500 - margin.top - margin.bottom;

    onMounted(() => {
      // pass ref with DOM element to D3, when mounted (DOM available)
      const svg = d3.select(svgRef.value);
      svg
        .attr("height", heightGraph + margin.top + margin.bottom)
        .attr("width", widthGraph + margin.right + margin.left);
      //this g is the container of graph
      var g = svg.select(".graph");
      g.attr("transform", "translate(" + margin.left + "," + margin.top + ")");
      //svg marker for arrow
      var defsarrow = g.append("svg:defs");
      //group of lines
      var allLinks = g.append("g").attr("class", "allLinkGroup");
      //group of nodes
      var allNodes = g.append("g").attr("class", "allNodeGroup");
      //group of text of lines
      var allTexts = g.append("g").attr("class", "allTextGroup");
      var link, node, linkText;
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
      //drag
      function dragstarted(d) {
        if (!d3.event.active) {
          simulation.alphaTarget(0.3).restart();
          (d.fx = d.x), (d.fy = d.y);
        }
      }
      function dragged(d) {
        d.fx = d3.event.x;
        d.fy = d3.event.y;
      }
      function dragended() {
        if (!d3.event.active) simulation.alphaTarget(0);
      }
      //mouse events
      const drag = d3
        .drag()
        .on("start", dragstarted)
        .on("drag", dragged)
        .on("end", dragended);
      //zoom on area graph
      svg.call(
        d3
          .zoom()
          .extent([
            [0, 0],
            [widthGraph, heightGraph],
          ])
          .scaleExtent([1, 10])
          .on("zoom", zoomed)
      );
      function zoomed() {
        g.attr("transform", d3.event.transform);
      }
      //active simulation
      var simulation = d3
        .forceSimulation()
        .force(
          "link",
          d3
            .forceLink()
            .id((d) => d.id)
            .distance(120)
        )
        // add some collision detection so they don't overlap
        .force("collide", d3.forceCollide().radius(30))

        //.nodes(refShape.value)
        .on("tick", ticked);
      //links
      link = allLinks
        .selectAll(".line")
        .data(refLink.value)
        .enter()
        .append("path") //for ticked
        .classed("line", true)
        .attr("marker-end", function () {
          return "url(#marker_" + "arrow" + ")";
        });
      //text of links
      linkText = allTexts
        .selectAll("text")
        .data(refLink.value)
        .enter()
        .append("text")
        .text((d) => d.text)
        .attr("class", "linetext");

      //node
      node = allNodes
        .selectAll(".node")
        .data(refShape.value)
        .enter()
        .append("g")
        .attr("class", "node")
        .call(drag);

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
      function update() {
        //Redefine and restart simulation
        simulation.nodes(refShape.value).on("tick", ticked);
        simulation.force("link").links(refLink.value);

        //update Links
        var linkU = allLinks.selectAll(".line").data(refLink.value),
          linkEnter = linkU
            .enter()
            .append("path")
            .classed("line", true)
            .attr("marker-end", function () {
              return "url(#marker_" + "arrow" + ")";
            });

        link = linkEnter.merge(linkU);
        link.exit().remove();

        var textU = allTexts.selectAll("text").data(refLink.value),
          textEnter = textU
            .enter()
            .append("text")
            .attr("class", "linetext")
            .text((d) => d.text);

        linkText = textEnter.merge(textU);
        linkText.exit().remove();

        var nodeU = allNodes.selectAll(".node").data(refShape.value),
          nodeEnter = nodeU
            .enter()
            .append("g")
            .call(drag)
            .attr("class", "node");
        nodeEnter
          .append("svg:rect")
          .attr("class", "rect")
          .attr("width", width + "px")
          .attr("height", height + "px");
        nodeEnter
          .append("text")
          .text((d) => d.text)
          .attr("text-anchor", "start")
          .attr("dominant-baseline", "auto")
          .attr("class", "nodetext")
          .attr("x", 0)
          .attr("y", 8);
        node = nodeEnter.merge(nodeU);
        node.exit().remove();
      }

      simulation.nodes(refShape.value).on("tick", ticked);
      simulation.force("link").links(refLink.value);
      function positionLink(d) {
        var offset = 60;

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
        return "translate(" + d.x + "," + d.y + ")";
      }

      function addNode(x) {
        console.log("node");
        refShape.value.push(x);
      }
      function addLink(x) {
        refLink.value.push(x);
      }
      svg.on("dblclick", () => {
        if(d3.event.ctrlKey){
          var position = d3.mouse(svg.node());
          addNode({
            id: refShape.value.length + 1,
            x: position[0],
            y: position[1],
            text: "New Node",
            index: null,
          });
          update();
          simulation.alphaTarget(0.3).restart();
        }
        
      });
      svg.on("click", () => {
        
          var new_link = {
            source: 5,
            target: 8,
            text: "Nuevo Link",
          };
          addLink(new_link);
          update();
          simulation.alphaTarget(0.3).restart();
        
      });
    });

    return { svgRef };
  },
};
</script>

<style>
.svgRef {
  background-color: #f7f6f2;
}
svg:hover {
  cursor: crosshair;
}
.node:hover {
  cursor: grab;
}
.graph {
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
