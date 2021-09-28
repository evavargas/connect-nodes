<template>
  <div style="padding: 2rem">
    <div ref="resizeRef" class="resizeRef">
      <svg ref="svgRef" class="svgRef">
        <g class="graph"></g>
      </svg>
    </div>
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
    const width = 68;
    const height = 46;
    //size for graph
    var widthGraph = 960;
    var heightGraph = 500;
    //elements for new link function
    var selected_node,
      selected_target_node,
      selected_link,
      new_line,
      should_drag = false,
      drawing_line = false;
    onMounted(() => {
      // pass ref with DOM element to D3, when mounted (DOM available)
      const svg = d3.select(svgRef.value);
      svg.attr("height", heightGraph).attr("width", widthGraph);
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
      var link, node, linkText;
      //to arrow
      defsarrow
        .selectAll("marker")
        .data(["end"])
        .enter()
        .append("svg:marker")
        .attr("id", "marker_arrow")
        .attr("markerHeight", 5)
        .attr("markerWidth", 5)
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
        console.log("dragstarted");
        if (!d3.event.active) {
          simulation.alphaTarget(0.2).restart();
          (d.fx = d.x), (d.fy = d.y);
        }
      }
      function dragged(d) {
        console.log("dragged");
        d.fx = d3.event.x;
        d.fy = d3.event.y;
      }
      function dragended() {
        console.log("dragended");
        if (!d3.event.active) simulation.alphaTarget(0);
      }
      //mouse events
      const drag = d3
        .drag()
        .filter(() => d3.event.ctrlKey || d3.event.metaKey)
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
      var simulation = d3
        .forceSimulation()
        .force(
          "link",
          d3
            .forceLink()
            .id((d) => d.id)
            .distance(140)
        )
        // add some collision detection so they don't overlap
        .force("collide", d3.forceCollide().radius(20));

      //mouse and key events
      d3.select(window)
        .on("mousemove", mousemove)
        .on("mouseup", mouseup)
        .on("keydown", keydown)
        .on("keyup", keyup);

      //context menu to add node
      svg.on("contextmenu", mousedown);

      function update() {
        link = allLinks
          .selectAll(".line")
          .data(refLink.value)
          .attr("d", positionLink)
          .classed("selected", (d) => d === selected_link);
        link
          .enter()
          .append("path")
          .attr("class", "line")
          .attr("marker-end", "url(#marker_arrow)")
          .on("mousedown", line_mousedown);
        link.exit().remove();

        linkText = allTexts
          .selectAll("text")
          .data(refLink.value)
          .text((d) => d.text);
        linkText.enter().append("text").attr("class", "linetext");
        linkText.exit().remove();

        node = allNodes
          .selectAll(".node")
          .data(refShape.value, (d) => d.id)
          .classed("selected", (d) => d === selected_node)
          .classed("selected_target", (d) => d === selected_target_node);
        var nodeg = node
          .enter()
          .append("g")
          .attr("class", "node")
          .call(drag)
          .attr("transform", (d) => "translate(" + d.x + "," + d.y + ")");

        nodeg
          .append("svg:rect")
          .attr("class", "rect")
          .attr("width", (width+16) + "px")
          .attr("height", (height+16) + "px")
          .on("mousedown", node_mousedown)
          .on("mouseover", node_mouseover)
          .on("mouseout", node_mouseout);
        nodeg
          .append("foreignObject")
          .html((d) => `<div style="width: ${width}px; height: ${height}px;">${d.text}</div>`)

          .attr("class", "nodetext")
          .attr("text-anchor", "start")
          .attr("x", 8)
          .attr("y", 8)
          .attr("width", width)
          .attr("height", height)
          .attr("dominant-baseline", "auto");
        //.text((d) => d.text)
        //
        //           .attr("x", 0)
        //           .attr("y", 2)
        //
        //
        //           .append("xhtml:body")
        //

        //           .attr("class", "nodetext")
        //           .attr("text-anchor", "start")

        //           .attr("dominant-baseline", "auto");
        node.exit().remove();

        simulation.nodes(refShape.value).on("tick", ticked);
        simulation.force(
          "link",
          d3
            .forceLink(refLink.value)
            .id((d) => d.id)
            .distance(135)
        );
        simulation.restart();
      }

      function ticked() {
        //curve the link
        link.attr("d", positionLink);

        //adding text to links
        linkText
          .attr("x", (d) => (d.source.x + d.target.x) / 2)
          .attr("y", (d) => (d.source.y + d.target.y) / 2);
        //position of nodes linked
        node.attr("transform", (d) => "translate(" + d.x + "," + d.y + ")");
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

      function addNode(x) {
        console.log("adding node");
        refShape.value.push(x);
      }
      function addLink(x) {
        console.log("adding link");
        refLink.value.push(x);
      }
      // select target node for new node connection
      function node_mouseover(d) {
        console.log("node_mouseover");
        if (drawing_line && d !== selected_node) {
          // highlight and select target node
          selected_target_node = d;
        }
      }

      function node_mouseout() {
        console.log("node_mouseout");
        if (drawing_line) {
          selected_target_node = null;
        }
      }

      // select node / start drag
      function node_mousedown(d) {
        console.log("node_mousedown");
        if (!drawing_line) {
          selected_node = d;
          selected_link = null;
        }
        if (!should_drag) {
          d3.event.stopPropagation();
          drawing_line = true;
        }
        d.fixed = true;
        simulation.stop();
        update();
      }

      // select line
      function line_mousedown(d) {
        console.log("line_mousedown");
        selected_link = d;
        selected_node = null;
        console.log(d);
        update();
      }

      // draw yellow "new connector" line
      function mousemove() {
        console.log("mousemove");
        if (drawing_line && !should_drag) {
          var m = d3.mouse(svg.node());
          var x = m[0];
          var y = m[1];
          // debounce - only start drawing line if it gets a bit big
          var dx = selected_node.x - x;
          var dy = selected_node.y - y;
          if (Math.sqrt(dx * dx + dy * dy) > 10) {
            // draw a line
            if (!new_line) {
              new_line = allLinks.append("line").attr("class", "new_line");
            }
            new_line
              .attr("x1", selected_node.x)
              .attr("y1", selected_node.y)
              .attr("x2", x)
              .attr("y2", y);
          }
        }
        update();
      }

      // add a new disconnected node
      function mousedown() {
        d3.event.preventDefault();
        console.log("mousedown");
        var m = d3.mouse(svg.node());
        addNode({
          id: refShape.value.length,
          x: m[0],
          y: m[1],
          text: "Text" + " " + refShape.value.length,
        });
        selected_link = null;
        simulation.stop();
        update();
        simulation.tick();
      }

      // end node select / add new connected node
      function mouseup() {
        console.log("mouseup");
        drawing_line = false;
        if (new_line) {
          if (selected_target_node) {
            selected_target_node.fixed = false;
            var new_node = selected_target_node;
          } else {
            var m = d3.mouse(svg.node());
            new_node = {
              id: refShape.value.length,
              x: m[0],
              y: m[1],
              text: "Text" + " " + refShape.value.length,
            };
            addNode(new_node);
          }
          selected_node.fixed = false;
          addLink({
            source: selected_node,
            target: new_node,
            text: selected_node.id + " to " + new_node.id,
          });
          selected_node = selected_target_node = null;
          update();
          setTimeout(function () {
            new_line.remove();
            new_line = null;
            simulation.tick();
          }, 300);
        }
      }

      // select for dragging node with shift; delete node with backspace
      function keydown() {
        console.log("keydown");
        switch (d3.event.keyCode) {
          case 8: // backspace
          case 46: {
            // delete
            if (selected_node) {
              console.log(selected_node);
              // deal with nodes
              var i = refShape.value.indexOf(selected_node);
              console.log(i);
              refShape.value.splice(i, 1);
              console.log(refShape.value);
              // find links to/from this node, and delete them too
              var new_links = [];
              refLink.value.forEach(function (l) {
                if (l.source !== selected_node && l.target !== selected_node) {
                  new_links.push(l);
                }
              });
              refLink.value = new_links;
              console.log(refLink.value);
              selected_node = refShape.value.length
                ? refShape.value[i > 0 ? i - 1 : 0]
                : null;
            } else if (selected_link) {
              console.log(selected_link);
              // deal with links
              i = refLink.value.indexOf(selected_link);
              console.log(i);
              refLink.value.splice(i, 1);
              console.log(refLink.value);
              selected_link = refLink.value.length
                ? refLink.value[i > 0 ? i - 1 : 0]
                : null;
            }
            update();
            console.log(refShape.value);
            console.log(refLink.value);
            break;
          }
          case 17: {
            // Ctrl
            should_drag = true;
            break;
          }
          case 91: {
            // Meta Right
            should_drag = true;
            break;
          }
          case 93: {
            // Meta left
            should_drag = true;
            break;
          }
        }
      }
      function keyup() {
        console.log("keyup");
        should_drag = false;
        simulation.tick();
      }
    });

    return { svgRef };
  },
};
</script>

<style>
.svgRef {
  background-color: #fcf0f0;
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
g.node .rect {
  fill: #f6e4a4;
  stroke: #cdbd82;
}
g.node .rect:hover{
  stroke-width: 2px;
}
.rect:active {
  cursor: grabbing;
}
.linetext {
  fill: #d3a16e;
}
.linetext:hover,
.nodetext {
  cursor: text;
}
.nodetext {
  fill: #8c866e;
  background-color: #FFDCB1;
}
.linetext,
.nodetext {
  font-size: 9px;
}
.line {
  fill: #ffffff00;
  stroke: #f9beb0;
  stroke-width: 2.4px;
}
.line:hover {
  cursor: pointer;
}
.arrow {
  fill: #cdbd82;
  stroke: #cdbd82;
}
.new_line {
  stroke: yellow;
  stroke-opacity: 0.8;
  stroke-width: 4px;
}

path.line.selected {
  stroke: red;
  stroke-opacity: 1;
}
g.node.selected rect.rect {
  fill: #f9b411;
  stroke-width: 1.5px;
}
g.node.selected_target rect.rect {
  fill: #f6f3a2;
  stroke-width: 1.5px;
}
</style>
