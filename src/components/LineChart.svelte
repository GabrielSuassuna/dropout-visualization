<script>
  import { onMount } from "svelte";
  import * as dc from "dc";
  import * as d3 from "d3";
  import crossfilter from "crossfilter2";

  export let facts;

  let title = "Taxas de evasão por ano - UFPE";

  const yearDim = facts.dimension((d) => d.NU_ANO_CENSO);

  const yearGroup = yearDim.group();

  console.log(yearGroup.all());

  const xScale = d3
    .scaleTime()
    .domain(d3.extent(yearDim.top(Infinity), (d) => d.dtg));

  let el;

  onMount(() => {
    let timeLineChart = dc.lineChart(el);

    timeLineChart
      .width(800)
      .height(150)
      .margins({ top: 10, right: 10, bottom: 20, left: 40 })
      .dimension(yearDim)
      .group(yearGroup)
      .x(xScale)
      .elasticY(true)
      .renderArea(false)
      .brushOn(true);

    timeLineChart.render();
  });
</script>

<div bind:this={el} class="chart">
  <h3>{title}</h3>
</div>
