<template>
  <div>
    <div class="canvas" id="canvas"></div>
  </div>
</template>

<script>
import * as d3 from "d3";
export default {
  name: "Chart",
  props: ["dotsData"],
  data() {
    return {
      svg:'',
    };
  },
  updated() {
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
          .data(this.dotsData)
          .join('circle')
          .attr("cx",d => d3.pointRadial(xScale(d.position[0]), yScale(d.position[1]) )[0])
          .attr("cy",d => d3.pointRadial(xScale(d.position[0]), yScale(d.position[1]) )[1])
          .classed("circle", true)
          .attr("transform", "translate(175,175)");

          //Adding the link paths 
          this.svg
          .selectAll("path")
          .data(this.dotsData)
          .join("path")
          .attr("d", link)
          .classed("link", true)
          .attr("transform", "translate(175,175)");

    // Adding text nodes, the x and y have special functions to get them in a place where they do not overlap the links
      this.svg
          .selectAll("text")
          .data(this.dotsData)
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

<style>
.canvas {
  background-color: #F0FFF3;
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
  fill: #FFFFFF00;
  stroke: darkmagenta;
}
</style>
