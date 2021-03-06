<template>
  <div ref="node"></div>
</template>

<script>
import * as d3 from "d3";
import fakeData from "@/utils/fakeHierarchyData";

export default {
  methods: {
    renderChart() {
      // 节点
      const node = this.$refs.node;
      // 清除旧数据
      d3.select(node)
        .select("svg")
        .remove();

      //画布大小
      const margin = { left: 90, top: 90, right: 90, bottom: 90 },
        width = 1000 - margin.left - margin.right,
        height = 1000 - margin.top - margin.bottom;

      const radius = width / 2;

      const color = d3.scaleOrdinal(d3.schemeCategory10);
      // 画布层
      const svg = d3
        .select(node)
        .append("svg")
        // 缩放
        .attr("preserveAspectRatio", "xMinYMin meet")
        .attr("viewBox", "0 0 " + width + " " + height)
        .style("width", "100%")
        .style("height", "auto");
      // 假数据
      const data = fakeData();

      const cluster = d3.cluster().size([2 * Math.PI, radius - 100]);
      const root = cluster(
        d3
          .hierarchy(data)
          .sort(
            (a, b) =>
              a.height - b.height || a.data.name.localeCompare(b.data.name)
          )
      );

      const layout = svg.attr("width", width).attr("height", height);

      const g = layout
        .append("g")
        .attr("transform", `translate(${width / 2}, ${height / 2})`);

      const link = g
        .append("g")
        .attr("fill", "none")
        .attr("stroke", "#555")
        .attr("stroke-opacity", 0.4)
        .attr("stroke-width", 1.5)
        .selectAll("path")
        .data(root.links())
        .enter()
        .append("path")
        .attr(
          "d",
          d3
            .linkRadial()
            .angle(d => d.x)
            .radius(d => d.y)
        );
      // 水平版
      // .attr(
      //   "d",
      //   d3
      //     .linkHorizontal()
      //     .x(function(d) {
      //       return d.y / 2;
      //     })
      //     .y(function(d) {
      //       return d.x / 2;
      //     })
      // );

      const nodes = g
        .append("g")
        .attr("stroke-linejoin", "round")
        .attr("stroke-width", 3)
        .selectAll("g")
        .data(root.descendants().reverse())
        .enter()
        .append("g")
        .attr(
          "transform",
          d => `
        rotate(${(d.x * 180) / Math.PI - 90})
        translate(${d.y},0)
      `
        );
      // 水平版
      // .attr("transform", d => `translate(${width / 2}, ${height / 2})`);

      nodes
        .append("circle")
        .attr("fill", d => (d.children ? "#555" : "#999"))
        .attr("r", 2.5);

      nodes
        .append("text")
        .attr("dy", "0.31em")
        .attr("x", d => (d.x < Math.PI === !d.children ? 6 : -6))
        .attr("text-anchor", d =>
          d.x < Math.PI === !d.children ? "start" : "end"
        )
        .attr("transform", d => (d.x >= Math.PI ? "rotate(180)" : null))
        .text(d => d.data.name)
        .filter(d => d.children)
        .clone(true)
        .lower()
        .attr("stroke", "white");
    }
  },
  mounted() {
    this.renderChart();
  },
  updated() {
    this.renderChart();
  }
};
</script>

<style>
</style>
