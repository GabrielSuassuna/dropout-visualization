<script>
  import { afterUpdate, onMount } from "svelte";
  import * as dc from "dc";
  import * as d3 from "d3";

  export let perfomanceDim;

  let title = "Taxas de evasão por ano - UFPE";

  const yearGroup = yearDim.group();

  const xScale = d3
    .scaleLinear()
    .domain(d3.extent(yearDim.top(Infinity), (d) => Number(d.NU_ANO_CENSO)));

  let el;

  afterUpdate(() => {
    let scatterPlotChart = dc.ScatterPlot(el);

    scatterPlotChart
      .width(768)
      .height(480)
      .x(d3.scaleLinear().domain([6, 20]))
      .brushOn(false)
      .symbolSize(8)
      .clipPadding(10)
      .yAxisLabel("This is the Y Axis!")
      .dimension(runDimension)
      .group(speedSumGroup);

    scatterPlotChart.render();
  });
</script>

<div bind:this={el} class="chart">
  <h3>{title}</h3>
</div>
