<template>
  <div id="canvas-dots">
  </div>
</template>

<script>
import * as d3 from "d3";

export default {
  name: "Canvas",
  //props: ["dotsData"],
  data() {
    return {
      canvas: {
        height: 500,
        width: 500,
      },
      svg: '',
      force: "",
      path: "",
      circle: '',
      g:'',
      drag:"",
      colourScale:'',
      nodes: [
        { id: 0,x:10, y:10, r:20, fill:'#FA4659', stroke:'#b7b7b7' },
        { id: 1,x:40, y:50, r:20, fill:'#FA4659', stroke:'#b7b7b7' },
        { id: 2,x:60, y:10, r:20, fill:'#FA4659', stroke:'#b7b7b7' },
      ],
      links: [{ source: "", target: "", in: false, out: true }],
    };
  },
  mounted() {
    this.selectSection();
    this.arrowLinks();
    this.appendNodes();
  },
  beforeUpdate() {},
  created() {
    //this.colourScale = d3.scaleOrdinal().range(['#614389', '#c5b9e6', '#b7b7b7']);
  },
  methods: {
    selectSection() {
      this.svg = d3
        .select("#canvas-dots")
        .append("svg")
        .attr("width", this.canvas.width)
        .attr("height", this.canvas.height)
      this.path = this.svg.append('svg:g').selectAll('path');
      this.circle = this.svg.append('svg:g').selectAll('g');
    },
    initForce() {
      // init D3 force layout
      this.force = d3
        .forceSimulation()
        .force("link", d3.forceLink()
        .id((d) => d.id).distance(150)
        )
        .force("charge", d3.forceManyBody().strength(-500))
        .force("x", d3.forceX(this.canvas.width / 2))
        .force("y", d3.forceY(this.canvas.height / 2))
        .on("tick", this.tick());
    },
    // update force layout (called automatically each iteration)
    tick() {
      // draw directed edges with proper padding from node centers
      this.path.attr("d", (d) => {
        const deltaX = d.target.x - d.source.x;
        const deltaY = d.target.y - d.source.y;
        const dist = Math.sqrt(deltaX * deltaX + deltaY * deltaY);
        const normX = deltaX / dist;
        const normY = deltaY / dist;
        const sourcePadding = d.left ? 17 : 12;
        const targetPadding = d.right ? 17 : 12;
        const sourceX = d.source.x + sourcePadding * normX;
        const sourceY = d.source.y + sourcePadding * normY;
        const targetX = d.target.x - targetPadding * normX;
        const targetY = d.target.y - targetPadding * normY;
        return `M${sourceX},${sourceY}L${targetX},${targetY}`;
      });
      this.circle.attr("transform", (d) => `translate(${d.x},${d.y})`);
    },
    arrowLinks() {
      // define arrow markers for graph links
      this.svg
        .append("svg:defs")
        .append("svg:marker")
        .attr("id", "end-arrow")
        .attr("viewBox", "0 -5 10 10")
        .attr("refX", 6)
        .attr("markerWidth", 3)
        .attr("markerHeight", 3)
        .attr("orient", "auto")
        .append("svg:path")
        .attr("d", "M0,-5L10,0L0,5")
        .attr("fill", "#000");

      this.svg
        .append("svg:defs")
        .append("svg:marker")
        .attr("id", "start-arrow")
        .attr("viewBox", "0 -5 10 10")
        .attr("refX", 4)
        .attr("markerWidth", 3)
        .attr("markerHeight", 3)
        .attr("orient", "auto")
        .append("svg:path")
        .attr("d", "M10,-5L0,0L10,5")
        .attr("fill", "#000");
    },
    appendNodes(){
      this.g =this.circle.enter().append('svg:g');
      for (let i = 0; i < this.nodes.length; i++) {
        this.g.append('svg:circle')
        .attr('class', 'node')
        .attr("cx", this.nodes[i].x)
        .attr('cy',this.nodes[i].y)
        .attr('r',this.nodes[i].r)
        .style('fill',this.nodes[i].fill)
        .style('stroke', this.nodes[i].stroke)
        
      }
      


    }
  },

  computed:{
      
  }
};
</script>

<style lang="css" scoped>
#canvas-dots{
    background-color: blue;
}
svg {
  background-color: #fff;
  cursor: default;
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  -o-user-select: none;
  user-select: none;
}

svg:not(.active):not(.ctrl) {
  cursor: crosshair;
}

path.link {
  fill: none;
  stroke: #000;
  stroke-width: 4px;
  cursor: default;
}

svg:not(.active):not(.ctrl) path.link {
  cursor: pointer;
}

path.link.selected {
  stroke-dasharray: 10, 2;
}

path.link.dragline {
  pointer-events: none;
}

path.link.hidden {
  stroke-width: 0;
}

circle.node {
  stroke-width: 1.5px;
  cursor: pointer;
}

circle.node.reflexive {
  stroke: #000 !important;
  stroke-width: 2.5px;
}

text {
  font: 12px sans-serif;
  pointer-events: none;
}

text.id {
  text-anchor: middle;
  font-weight: bold;
}
</style>