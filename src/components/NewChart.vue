<template>
  <button v-on:click="addData">New Node</button>
  <div class="canvasData" id="canvasData" v-if="datashapes"></div>
</template>

<script>
import { reactive } from "vue";
import * as d3 from "d3";
export default {
  name: "NewChart",
  props: { datashapes: Array, datalinks: Array },
  data() {
    return {
      width: "600",
      height: "600",
      svg: "",
      newNode: {
        id: 0,
        y: 30,
        x: 30,
        text: "hola",
        vy: 4,
        vx: 4,
        index: 0,
      },
    };
  },
  async created() {},
  mounted() {
    this.initialize();
  },
  methods: {
    addData() {
      let newNode = this.newNode;
      newNode.id = this.shapes.length;
      newNode.index = newNode.id;
      this.shapes.push(newNode);
      console.log(this.shapes)
    },
    //selecting area
    initialize() {
      // asign data to internal vars
      var lines = this.links;
      var square = this.shapes;
      //setting area of graph
      this.width = window.innerWidth * 0.66;
      this.height = window.innerHeight * 0.5;

      //selecting area to graph
      this.svg = d3
        .select("#canvasData")
        .append("svg")
        .attr("viewBox", [0, 0, this.width, this.height])
        .attr("transform", "translate(0,00)");

      //this g is the container of graph
      var g = this.svg.append("g").attr("cursor", "grab");
     
      var link, linkEnter, node, nodeEnter;


      // linking elements with force simulation
      var simulation = d3
        .forceSimulation()
        .force("link", d3 .forceLink().id((d) => d.id).distance(100))
        .on("tick", tick);

      //group of lines
      var allLinks = g.append("g")
      .attr("class", "allLinkGroup")
      
      //group of nodes
      var allNode = g.append("g")
      .attr("class", "allnodeGroup")
      update()
      simulation.restart()

      //update simulation
      function update(){
        //links update
        link = allLinks.selectAll(".line")
        .data(lines)
        .attr("transform", "translate(150,150)");
        link.exit().remove()

         //attaching lines to graph
        linkEnter = link.enter()
        .append("line")
        .attr("class", "line")
        .attr("transform", "translate(150,150)");

        link = linkEnter.merge(link).attr("class", "line");

        //nodes updates
        node =allNode.selectAll(".node")
        .data(square)
        .attr("transform", "translate(150,150)")

        //attaching nodes to graph
        nodeEnter = node.enter().append("g").attr("class", "node")

        //joinning square to nodes
        nodeEnter.append("svg:rect").attr("class", "rect")
        .attr("transform", "translate(150,150)")

        //joinning text to node
        nodeEnter.append("text")
        .text((d)=> d.text)
        .attr("text-anchor", "middle")
        .attr("dominant-baseline", "end")
        .attr("font-size", "8px")
        .attr("x", (d) => d.x + 15)
        .attr("y", (d) => d.y + 10)
        .attr("transform", "translate(150,150)");

        node = nodeEnter.merge(node)
        node.exit().remove();

        simulation.nodes(square);
        simulation.force("link", d3 .forceLink(lines).id((d) => d.id).distance(100))

      }
      
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
        simulation.alpha(1).restart();
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
      this.svg.call(
        d3.zoom().extent([[0, 0],[this.width, this.height],])
          .scaleExtent([1, 8])
          .on("zoom", zoomed)
      );
      function zoomed() {
        console.log(g);
        g.attr("transform", d3.event.transform);
      }

   //button to addNode
      var button = this.svg.append("rect");
      button.attr("class","div").html("new node")
      button.on("click", this.addData);


      //publish graph
      return this.svg.node();
    },
  },
  computed: {
    shapes: {
      get() {
        return reactive(this.datashapes);
      },
      set(newValue) {
        this.datashapes.concat(newValue);
      },
    },
    links: {
      get() {
        return reactive(this.datalinks);
      },
      set(newValue) {
        this.datalinks.concat(newValue);
      },
    },
  },
};
</script>

<style>
.canvasData {
  background-color: #eeeeee;
}
.div{
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
  display: inline;
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
