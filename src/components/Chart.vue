<template>
  <div style="padding: 2rem; margin-top: 56px">
    <div ref="resizeRef" class="resizeRef">
      <svg ref="svgRef" class="svgRef"></svg>
      <div class="menu-inner box-inner">
        <ul>
          <li>Add a node: right click. (Context menu)</li>
          <li>
            Link nodes: drag from a side-node to another to connect nodes.
          </li>
          <li>Additionaly if not target is selected, It creates a new node</li>
        </ul>
        <ul>
          Drag and drop
          <li>Windows: Ctrl + left click.</li>
          <li>Mac: ⌘+ left click.</li>
        </ul>
        <ul>
          <li>Remove nodes: Press Supr / Backspace to delete nodes or lines</li>
        </ul>
        <ul>
          Also, use the buttons
        </ul>
         <div>
           <button class="inner-btn addnode">Add node</button>
           <button class="inner-btn delelem">Delete element</button>
         </div>
      </div>
    </div>
  </div>
</template>

<script>
import { onMounted, ref } from "vue";
import * as d3 from "d3";
import mousedown from "./Chart.vue";

export default {
  name: "Chart",
  props: { datashapes: Array, datalinks: Array },
  setup(props) {
    // create ref to pass to D3 for DOM manipulation
    const resizeRef = ref(null);
    const svgRef = ref(null);
    var refShape = ref(props.datashapes);
    var refLink = ref(props.datalinks);
    //width and heigt for rect of nodes
    const width = 96;
    const height = 78;
    //size for graph
    var widthGraph = 1000;
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
      var contGraph = svg.append("rect");
      contGraph
        .attr("height", heightGraph)
        .attr("width", widthGraph)
        .attr("class", "container-graph");

      //this g is the container of graph
      var g = svg.append("g").attr("class", "graph");
      var gmenu = d3.select(resizeRef.value);
      var btnAddNode = gmenu.select(".addnode");
      btnAddNode.on("mousedown", function(){
        addingNode();
      })
      var btnDelElem = gmenu.select(".delelem");
      btnDelElem.on("mousedown", function(){
        deleteNodeorLink();
      })
      

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
        if (!d3.event.active) {
          simulation.alphaTarget(0.2).restart();
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
        .filter(() => d3.event.ctrlKey || d3.event.metaKey)
        .on("start", dragstarted)
        .on("drag", dragged)
        .on("end", dragended);
      //zoom on area graph
      contGraph.call(
        d3
          .zoom()
          .on("zoom", zoomed)
          .extent([
            [0, 0],
            [widthGraph, heightGraph],
          ])
          .scaleExtent([1, 10])
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
            .distance(160)
        )
        // add some collision detection so they don't overlap
        .force("collide", d3.forceCollide().radius(10));

      //mouse and key events
      d3.select(window)
        .on("mousemove", mousemove)
        .on("mouseup", mouseup)
        .on("keydown", keydown)
        .on("keyup", keyup);

      //context menu to add node
      contGraph.on("contextmenu", mousedown).on("click", () => {
        d3.event.preventDefault();
        selected_node = null;
        selected_link = null;
        simulation.stop();
        update();
        simulation.tick();
      });

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

        linkText = allTexts.selectAll(".linetextbody").data(refLink.value);
        var linkTextg = linkText
          .enter()
          .append("svg:foreignObject")
          .attr("class", "linetextbody")
          .attr("value", (d) => `${d.id}`)
          .attr("width", 82 + "px")
          .attr("height", 21 + "px");
        linkTextg
          .html(
            (d) =>
              `<textarea class="textarea-line" id="linetext${d.id}">${d.text}</textarea>`
          )
          .attr("x", 3)
          .attr("dominant-baseline", "auto")
          .on("change", updateTextLink);
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
          .attr("value", (d) => `${d.id}`)
          .call(drag)
          .attr("transform", (d) => "translate(" + d.x + "," + d.y + ")");

        nodeg
          .append("svg:rect")
          .attr("class", "rect")
          .attr("width", width + "px")
          .attr("height", height + "px")
          .attr("rx", "5")
          .attr("ry", "5")
          .on("mousedown", node_mousedown)
          .on("mouseover", node_mouseover)
          .on("mouseout", node_mouseout);
        nodeg
          .append("foreignObject")
          .attr("x", 7)
          .attr("y", 7)
          .attr("class", "nodebody")
          .attr("width", width - 16 + "px")
          .attr("height", height - 13 + "px")
          .html(
            (d) =>
              `<textarea class="textarea-node" id="nodetext${d.id}">${d.text}</textarea>`
          )
          .attr("dominant-baseline", "auto")
          .on("change", updateTextNode);
        node.exit().remove();

        simulation.nodes(refShape.value).on("tick", ticked);
        simulation.force(
          "link",
          d3
            .forceLink(refLink.value)
            .id((d) => d.id)
            .distance(160)
        );
        simulation.restart();
      }

      function ticked() {
        //curve the link
        link.attr("d", positionLink);

        //adding text to links
        linkText
          .attr(
            "x",
            (d) => (d.source.x + width / 2 + (d.target.x + width / 2)) / 2 - 15
          )
          .attr("y", (d) => (d.source.y + height + d.target.y) / 2 - 20);
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
        refShape.value.push(x);
      }
      function addLink(x) {
        refLink.value.push(x);
      }
      function editTextNode(textAreaId, textArea) {
        let i = parseInt(
          document
            .getElementById(textAreaId)
            .parentNode.parentNode.getAttribute("value")
        );
        let element = refShape.value.filter((e) => e.id == i)[0]; //data from element that was edited
        element.text = textArea.innerHTML;
      }
      function editTextLink(textAreaId, textArea) {
        let i = parseInt(
          document.getElementById(textAreaId).parentNode.getAttribute("value")
        );
        let element = refLink.value.filter((e) => e.id == i)[0]; //data from element that was edited
        element.text = textArea.innerHTML;
      }
      function updateTextNode() {
        let textAreaId = d3.select(this).select(".textarea-node").attr("id"); //element Id
        let textArea = document.getElementById(textAreaId); //element
        textArea.innerHTML = textArea.value;
        d3.select(this).select(".textarea-node").html(textArea.innerHTML); //end on change
        textArea.onblur = function () {
          textArea.innerHTML = textArea.value;
          d3.select(this).select(".textarea-node").html(textArea.innerHTML); //end on blur
        };
        editTextNode(textAreaId, textArea);
      }
      function updateTextLink() {
        let textAreaId = d3.select(this).select(".textarea-line").attr("id"); //element Id
        let textArea = document.getElementById(textAreaId); //element
        textArea.innerHTML = textArea.value;
        d3.select(this).select(".textarea-node").html(textArea.innerHTML); //end on change
        textArea.onblur = function () {
          textArea.innerHTML = textArea.value;
          d3.select(this).select(".textarea-node").html(textArea.innerHTML); //end on blur
        };
        editTextLink(textAreaId, textArea);
      }
      // select target node for new node connection
      function node_mouseover(d) {
        if (drawing_line && d !== selected_node) {
          // highlight and select target node
          selected_target_node = d;
        }
      }

      function node_mouseout() {
        if (drawing_line) {
          selected_target_node = null;
        }
      }

      // select node / start drag
      function node_mousedown(d) {
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
        selected_link = d;
        selected_node = null;
        update();
      }

      // draw yellow "new connector" line
      function mousemove() {
        if (drawing_line && !should_drag) {
          var m = d3.mouse(g.node());
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
      function addingNode() {
        // d3.event.preventDefault();
        //var m = d3.mouse(svg.node());
        addNode({
          id: refShape.value.length,
          x: 10,
          y: 10,
          text: "Text" + " " + refShape.value.length,
        });
        selected_link = null;
        simulation.stop();
        update();
        simulation.tick();
      }
      function mousedown() {
        d3.event.preventDefault();
        var m = d3.mouse(g.node());
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
        drawing_line = false;
        if (new_line) {
          if (selected_target_node) {
            selected_target_node.fixed = false;
            var new_node = selected_target_node;
          } else {
            var m = d3.mouse(g.node());
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
            id: refLink.value.length,
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
          }, 220);
        }
      }
      function deleteNodeorLink() {
        if (selected_node) {
          // deal with nodes
          var i = refShape.value.indexOf(selected_node);
          refShape.value.splice(i, 1);
          // find links to/from this node, and delete them too
          var new_links = [];
          refLink.value.forEach(function (l) {
            if (l.source !== selected_node && l.target !== selected_node) {
              new_links.push(l);
            }
          });
          refLink.value = new_links;
          selected_node = refShape.value.length
            ? refShape.value[i > 0 ? null : null]
            : null;
          simulation.stop();
          update();
          simulation.tick();
        } else if (selected_link) {
          // deal with links
          i = refLink.value.indexOf(selected_link);
          refLink.value.splice(i, 1);
          selected_link = refLink.value.length
            ? refLink.value[i > 0 ? null : null]
            : null;
          simulation.stop();
          update();
          simulation.tick();
        }
      }

      // select for dragging node with shift; delete node with backspace
      function keydown() {
        switch (d3.event.keyCode) {
          case 8: // backspace
          case 46: {
            // delete
            if (selected_node) {
              // deal with nodes
              var i = refShape.value.indexOf(selected_node);
              refShape.value.splice(i, 1);
              // find links to/from this node, and delete them too
              var new_links = [];
              refLink.value.forEach(function (l) {
                if (l.source !== selected_node && l.target !== selected_node) {
                  new_links.push(l);
                }
              });
              refLink.value = new_links;
              selected_node = refShape.value.length
                ? refShape.value[i > 0 ? null : null]
                : null;
            } else if (selected_link) {
              // deal with links
              i = refLink.value.indexOf(selected_link);
              refLink.value.splice(i, 1);
              selected_link = refLink.value.length
                ? refLink.value[i > 0 ? null : null]
                : null;
            }
            update();
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
        should_drag = false;
        simulation.tick();
      }
      update();
      simulation.restart();
      update();
    });

    return { svgRef, mousedown, resizeRef };
  },
};
</script>

<style>
.svgRef {
  box-shadow: 5px 10px #888888;
}
.container-graph {
  fill: #fcf0f064;
}
.container-graph:hover {
  cursor: crosshair;
}
.node:hover {
  cursor: grab;
}
.graph {
  cursor: move;
}
g.node .rect {
  fill: #fed17e;
  stroke: #f3c371;
}
g.node .rect:hover {
  stroke-width: 3px;
}
.rect:active {
  cursor: grabbing;
}

.linetextbody:hover,
.nodebody {
  cursor: text;
}

.linetextbody,
.nodebody {
  background-color: #fbd594;
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
  fill: #f3c371;
  stroke-width: 1.5px;
}
g.node.selected_target rect.rect {
  fill: #f6f3a2;
  stroke-width: 1.5px;
}
.textarea-node {
  background-color: #fbd594;
  font-size: 9px;
  width: 80px;
  height: 64px;
  resize: none;
  overflow: hidden;
  padding: 0;
  border: none;
  text-anchor: start;
}
.textarea-line {
  background-color: #fbd594;
  font-size: 9px;
  width: 80px;
  resize: none;
  overflow: hidden;
  padding: 0;
  border: none;
  text-anchor: start;
  margin-top: 1px;
}
.linetextbody {
  background-color: #fed17e;
  border-radius: 4px;
}
.box-inner {
  background-color: #fff;
  width: 250px;
  position: absolute;
  padding: 0px 2rem;
  display: inline-block;
  justify-content: left;
  align-items: left;
  box-shadow: 0 1px 1px rgba(0, 0, 0, 0.16), 0 1px 1px rgba(0, 0, 0, 0.23);
  margin-left: 0.7rem;
}
.box-inner {
  margin: 0;
  font-size: 14px;
  font-family: Helvetica, sans-serif;
}
.box-inner button {
  margin: 0.7rem;
  width: 100px;
  font-size: 12px;
}
</style>
