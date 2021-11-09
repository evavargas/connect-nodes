<template>
  <div class="mainChart">
    <div ref="resizeRef" class="resizeRef">
      
      <div class="box-inner">
        <button class="inner-btn addnode">Add <br>node</button>
        <button class="inner-btn delelem">Delete<br> element</button>
      </div>
      <svg ref="svgRef" class="svgRef"></svg>
    </div>
  </div>
</template>

<script>
import { onMounted, ref } from "vue";
import * as d3 from "d3";

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
    const width = 100;
    const height = 62.5;
    //size for graph
    var widthGraph = 1220;
    var heightGraph = 728;
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
      btnAddNode.on("mousedown", function () {
        addingNode();
      });
      var btnDelElem = gmenu.select(".delelem");
      btnDelElem.on("mousedown", function () {
        deleteNodeorLink();
      });

      //svg marker for arrow
      var defsarrow = g.append("svg:defs");
      //group of text of lines
      var allTexts = g.append("g").attr("class", "allTextGroup");
      //group of nodes
      var allNodes = g.append("g").attr("class", "allNodeGroup");

      var node, linkText;
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
        if (!d3.event.active) simulation.alphaTarget(0.2);
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
      var simulation = d3.forceSimulation();
      simulation.force("collide", d3.forceCollide().radius(10));
      simulation.nodes(refShape.value).on("tick", ticked);
      simulation.force(
        "link",
        d3
          .forceLink(refLink.value)
          .id((d) => d.id)
          .distance(130)
      );
      simulation.tick();
      // add some collision detection so they don't overlap

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
        update();
        simulation.tick();
      });

      function update() {
        //update function join the data with the graph elements
        node = allNodes
          .selectAll(".node")
          .data(refShape.value, (d) => d.id)
          .classed("selected", (d) => d === selected_node)
          .classed("selected_target", (d) => d === selected_target_node)
          .on("mousedown", node_mousedown)
          .on("mouseover", node_mouseover)
          .on("mouseout", node_mouseout);
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
          .attr("ry", "5");

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

        linkText = allTexts
          .selectAll(".linkWText")
          .data(refLink.value, (d) => d.id)
          .classed("selected", (d) => d === selected_link);
        var linkTextg = linkText.enter().append("g").attr("class", "linkWText");
        linkTextg
          .append("svg:path")
          .attr("class", "line")
          .attr("marker-end", "url(#marker_arrow)")
          .on("mousedown", line_mousedown);
        linkTextg
          .append("svg:foreignObject")
          .attr("class", "linetextbody")
          .attr("value", (d) => `${d.id}`)
          .attr("width", 82 + "px")
          .attr("height", 21 + "px")
          .html(
            (d) =>
              `<textarea class="textarea-line" id="linetext${d.id}">${d.text}</textarea>`
          )
          .attr("dominant-baseline", "auto")
          .on("change", updateTextLink);
        linkText.exit().remove();
        simulation.restart();
      }

      function ticked() {
        //ticked give the position to the elements on the simluation graph
        node.attr("transform", (d) => "translate(" + d.x + "," + d.y + ")");
        //curve the link
        allTexts.selectAll(".line").attr("d", positionLink);
        allTexts
          .selectAll(".linetextbody")
          .attr(
            "x",
            (d) => (d.source.x + width / 2 + (d.target.x + width / 2)) / 2 - 15
          )
          .attr("y", (d) => (d.source.y + height + d.target.y) / 2 - 20);
      }
      function positionLink(d) {
        //this function is the curve of the links
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
      function getIdNode() {
        //asign an Id for a node
        if (refShape.value.length == 0) {
          return 0;
        }
        let listOfIds = refShape.value.map((x) => x.id);
        return Math.max.apply(null, listOfIds) + 1;
      }
      function getIdLink() {
        //asign an Id for a link
        if (refLink.value.length == 0) {
          return 0;
        }
        let listOfIds = refLink.value.map((x) => x.id);
        return Math.max.apply(null, listOfIds) + 1;
      }

      function addNode(x) {
        refShape.value.push(x);
      }
      function addLink(x) {
        refLink.value.push(x);
      }

      function updateTextNode() {
        let textAreaId = d3.select(this).select(".textarea-node").attr("id"); //element id : nodetext${d.id}
        let textArea = document.getElementById(textAreaId); //element with the id
        textArea.innerHTML = textArea.value; //updating text inside
        d3.select(this).select(".textarea-node").html(textArea.innerHTML); //updating text in html doc
        textArea.onblur = function () {
          //updating text on blur event
          textArea.innerHTML = textArea.value; //updating text inside
          d3.select(this).select(".textarea-node").html(textArea.innerHTML); //updating text in html doc
        };
        editTextNode(textAreaId, textArea);
      }
      ///////////////////////////
      function editTextNode(textAreaId, textArea) {
        let valueid = parseInt(
          document
            .getElementById(textAreaId)
            .parentNode.parentNode.getAttribute("value") //value: ${d.id}
        );
        let element = refShape.value.filter((e) => e.id == valueid)[0]; //data from element that was edited
        element.text = textArea.innerHTML;
      }

      ////////////////////////////
      function updateTextLink() {
        let textAreaId = d3.select(this).select(".textarea-line").attr("id"); //element id : linetext${d.id}
        let textArea = document.getElementById(textAreaId); //element with the id
        textArea.innerHTML = textArea.value; //updating text inside
        d3.select(this).select(".textarea-line").html(textArea.innerHTML); //updating text in html doc
        textArea.onblur = function () {
          //updating text on blur event
          textArea.innerHTML = textArea.value; //updating text inside
          d3.select(this).select(".textarea-line").html(textArea.innerHTML); //updating text in html doc
        };
        editTextLink(textAreaId, textArea);
      }
      ///////////////////////////////
      function editTextLink(textAreaId, textArea) {
        let i = parseInt(
          document.getElementById(textAreaId).parentNode.getAttribute("value") //value: ${d.id}
        );
        let element = refLink.value.filter((e) => e.id == i)[0]; //data from element that was edited
        element.text = textArea.innerHTML;
      }
      //////////////////////////////

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
              new_line = allTexts.append("line").attr("class", "new_line");
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

      // add a new disconnected node in a specific position
      function addingNode() {
        addNode({
          id: getIdNode(),
          x: 10,
          y: 10,
          fx: 10,
          fy: 10,
          text: "Text" + " " + getIdNode(),
          fixed: true,
        });
        selected_link = null;
        simulation.stop();
        update();
        simulation.tick();
      }
      function mousedown() {
        // add a new disconnected node in the mouse position
        d3.event.preventDefault();
        var m = d3.mouse(g.node());
        addNode({
          id: getIdNode(),
          x: m[0],
          y: m[1],
          fx: m[0],
          fy: m[1],
          text: "Text" + " " + getIdNode(),
          fixed: true,
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
              id: getIdNode(),
              x: m[0],
              y: m[1],
              fx: m[0],
              fy: m[1],
              text: "Text" + " " + getIdNode(),
              fixed: true,
            };
            addNode(new_node);
          }
          selected_node.fixed = false;
          addLink({
            id: getIdLink(),
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
          if (refShape.value.indexOf(selected_node) > -1) {
            let i = refShape.value.indexOf(selected_node);
            refShape.value.splice(i, 1);
          }
          // find links to/from this node, and delete them too

          let new_links = [];
          refLink.value.forEach((l) => {
            if (l.source !== selected_node && l.target !== selected_node) {
              new_links.push(l); //the list of links not related with the selected node
            }
          });
          refLink.value.splice(0, refLink.value.length);
          refLink.value.splice(0, 1); // clean the list of links
          for (let i = 0; i < new_links.length; i++) {
            refLink.value.push(new_links[i]);
          }
          selected_node = null;
        } else if (selected_link) {
          // deal with links
          let i = refLink.value.indexOf(selected_link);
          refLink.value.splice(i, 1);
          selected_link = null;
        }
        update();
      }

      // select for dragging node with shift; delete node with backspace
      function keydown() {
        switch (d3.event.keyCode) {
          case 8: // backspace
          case 46: {
            // delete
            deleteNodeorLink();
            simulation.restart();
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

    return { svgRef, resizeRef };
  },
};
</script>

<style>
.mainChart{
  margin-top: 39.7px;
  padding: 0 6px;
}
.svgRef {
  box-shadow: -2px 5px 10px #888888;
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
g.node .rect {
  fill: #fed17e;
  stroke: #f3c371;
}
g.node .rect:hover {
  stroke-width: 5px;
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

g.selected path.line {
  stroke: rgb(237, 97, 97);
  stroke-opacity: 1;
}
.line:hover {
  stroke: #f3c371;
  stroke-width: 3px;
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
  width: fit-content;
  width: -webkit-fit-content;
  width: -moz-fit-content;
  top: 39px;
  right: 0.5rem;
  z-index: 1;
  position: fixed;
  display: inline-block;
  box-shadow: 0 1px 1px rgba(0, 0, 0, 0.16), 0 1px 1px rgba(0, 0, 0, 0.23);
}

.box-inner button {
  display: block;
  margin: 0.7rem;
  width: 80px;
  font-size: 11px;
}
</style>
