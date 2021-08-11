<template>
  <div>
    <h1>Nodes linked with D3</h1>
    <h2>{{ msg }}</h2>
    <div class="canvas" id="canvas"></div>
  </div>
</template>

<script>
import * as d3 from "d3";
export default {
  name: "PackChart",
  props: ["dotsData"],
  data() {
    return {
      msg: "😃",
      svg:'',
      nodes:[
    {id: "HI",    position: [2, 0], parentPosition: [2, 0]},
    {id: "I",     position: [1, 1], parentPosition: [2, 0]},
    {id: "Am",    position: [3, 1], parentPosition: [2, 0]},
    {id: "A",     position: [0, 2], parentPosition: [1, 1]},
    {id: "Graph", position: [1, 2], parentPosition: [1, 1]},
    {id: "With",  position: [2, 2], parentPosition: [1, 1]},
    {id: "Nodes", position: [3, 2], parentPosition: [3, 1]},
    {id: "and",   position: [4, 2], parentPosition: [3, 1]},
    {id: "Links", position: [5, 2], parentPosition: [3, 1]}

      ],
    };
  },
  mounted() {
    this.chart();
  },
  methods: {
    chart() {
      this.svg = d3.select("#canvas").append("svg").attr("viewBox", [0, 0, 600, 600]);
      var xScale = d3.scaleLinear().domain([0, 8]).range([0, Math.PI * 2]);
      var yScale = d3.scaleLinear().domain([0,2]).range([0, 133]);

      //New radial link generator
      var link = d3.linkRadial()
        .source(d => d.position)
        .target(d => d.parentPosition)
        .angle( d => xScale(d[0]))
        .radius( d => yScale(d[1]));

        //Agregando nodos al canvas
        this.svg
          .selectAll("circle")
          .data(this.nodes)
          .join('circle')
          .attr("cx",d => d3.pointRadial(xScale(d.position[0]), yScale(d.position[1]) )[0])
          .attr("cy",d => d3.pointRadial(xScale(d.position[0]), yScale(d.position[1]) )[1])
          .classed("circle", true)
          .attr("transform", "translate(175,175)");

          //Adding the link paths 
          this.svg
          .selectAll("path")
          .data(this.nodes)
          .join("path")
          .attr("d", link)
          .classed("link", true)
          .attr("transform", "translate(175,175)");

    // Adding text nodes, the x and y have special functions to get them in a place where they do not overlap the links
      this.svg
          .selectAll("text")
          .data(this.nodes)
          .join("text")
          .attr("font-size", "11px")
          .attr("text-anchor", "middle")
          .attr("x", function(d) {
            if(d.position[1] == 2)
                return d3.pointRadial(xScale(d.position[0]), yScale(d.position[1]) + 25)[0];
              if(d.position[1] == 1){
                  var xPos = d3.pointRadial(xScale(d.position[0]), yScale(d.position[1]))[0];
                  xPos = xPos > 0 ? xPos - 25 : xPos + 15;
                  return xPos;
              }                   
              return 0;})
          .attr("y", function(d){
              return d.position[1] == 2 ? d3.pointRadial(xScale(d.position[0]), yScale(d.position[1]) + 20)[1] + 4 :
                       d.position[1] == 1 ? d3.pointRadial(xScale(d.position[0]) + .15, yScale(d.position[1]) + 10)[1] :
                                    20 
                  })
          .text(d => d.id)
          .attr("transform", "translate(175,175)");
          return this.svg.node();
    },
  },
  computed: {},
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
.canvas {
  background-color: #F0FFF3;
  width: 100%;
  height: 100%;
}
.circle{
  background-color: blue;
  r: 5px;
  fill: blueviolet;
  stroke: brown;
}
svg{
  display: inline;
}
.link{
  fill: cyan;
  stroke: darkmagenta;
}
</style>
