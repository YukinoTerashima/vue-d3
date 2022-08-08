<template>
  <div>
    <input v-model="minValue" placeholder="Minimum value" />
    <p>Minimum value is {{ minValue }}</p>
  </div>
  <div id="bar-chart"></div>
</template>
<script lang="ts">
import { ref, onMounted, watch } from "vue";
import * as d3 from "d3";

type Dict = { key: string; value: number; code: string };

export default {
  name: "Chart",
  setup() {
    const minValue = ref(0);
    const data = ref([
      { key: "United States", value: 336, code: "USA" },
      { key: "United Kingdom", value: 98, code: "GBR" },
      { key: "Germany", value: 79, code: "DEU" },
      { key: "France", value: 60, code: "FRA" },
      { key: "Sweden", value: 29, code: "SWE" },
      { key: "Switzerland", value: 23, code: "CHE" },
      { key: "Japan", value: 21, code: "JPN" },
      { key: "Russia", value: 19, code: "RUS" },
      { key: "Netherlands", value: 17, code: "NLD" },
      { key: "Austria", value: 14, code: "AUT" },
    ]);

    const renderChart = (data: Array<Dict>, minValue: Number) => {
      // Copyright 2021 Observable, Inc.
      // Released under the ISC license.
      // https://observablehq.com/@d3/bar-chart
      // Compute values.
      const X = d3.filter(data, (d) => d.value >= minValue).map((d) => d.code);
      const Y = d3.filter(data, (d) => d.value >= minValue).map((d) => d.value);

      const xDomain = new d3.InternSet(X);
      const yDomain = [0, d3.max(Y) || 0];

      // Omit any data not present in the x-domain.
      const I = d3.range(X.length).filter((i) => xDomain.has(X[i]));

      // Construct scales, axes, and formats.
      const height = 400;
      const width = 400;
      const marginBottom = 30;
      const marginTop = 20;
      const marginLeft = 40;
      const marginRight = 0;
      const xRange = [marginLeft, width - marginRight];
      const xPadding = 0.1;
      const yRange = [height - marginBottom, marginTop];
      const yLabel = "Frequency";

      const xScale = d3.scaleBand(xDomain, xRange).padding(xPadding);
      const yScale = d3.scaleLinear(yDomain, yRange);
      const xAxis = d3.axisBottom(xScale).tickSizeOuter(0);
      const yAxis = d3.axisLeft(yScale).ticks(10);
      const color = "steelblue";

      // Compute titles.
      const formatValue = yScale.tickFormat(100);
      const title = (i: number) => `${X[i]}\n${formatValue(Y[i])}`;


      const chartHolder = d3.select("#bar-chart");

      const svg = chartHolder
        .append("svg")
        .attr("width", width)
        .attr("height", height)
        .attr("viewBox", [0, 0, width, height])
        .attr("style", "max-width: 100%; height: auto; height: intrinsic;");

      svg
        .append("g")
        .attr("transform", `translate(${marginLeft},0)`)
        .call(yAxis)
        .call((g) => g.select(".domain").remove())
        .call((g) =>
          g
            .selectAll(".tick line")
            .clone()
            .attr("x2", width - marginLeft - marginRight)
            .attr("stroke-opacity", 0.1)
        )
        .call((g) =>
          g
            .append("text")
            .attr("x", -marginLeft)
            .attr("y", 10)
            .attr("fill", "currentColor")
            .attr("text-anchor", "start")
            .text(yLabel)
        );

      const bar = svg
        .append("g")
        .attr("fill", color)
        .selectAll("rect")
        .data(I)
        .join("rect")
        .attr("x", (i) => xScale(X[i]))
        .attr("y", (i) => yScale(Y[i]))
        .attr("height", (i) => yScale(0) - yScale(Y[i]))
        .attr("width", xScale.bandwidth());

      if (title) bar.append("title").text(title);

      svg
        .append("g")
        .attr("transform", `translate(0,${height - marginBottom})`)
        .call(xAxis);

      return svg.node();
    };

    onMounted(() => {
      renderChart(data.value, minValue.value);
    });

    watch(minValue, (minValue) => {
      d3.select("svg").remove()
      renderChart(data.value, minValue);
    });


    return {
      renderChart,
      minValue,
    };
  },
};
</script>
<style>
#bar-chart {
  width: 600px;
  height: 400px;
}
.axis {
  font: 10px sans-serif;
}
.axis path,
.axis line {
  fill: none;
  stroke: #000;
  shape-rendering: crispEdges;
}
</style>
