<script>
  import { onMount } from "svelte";
  import * as dc from "dc";
  import * as d3 from "d3";

  export let yearDim;

  let title = "Número de evasões de 2017 a 2019 - UFPE";

  const yearGroup = yearDim.group().reduceCount();

  const xScale = d3
    .scaleTime()
    .domain(d3.extent(yearDim.top(Infinity), (d) => d.NU_ANO_CENSO));

  let el;

  onMount(() => {
    let timeLineChart = dc.lineChart(el);

    timeLineChart
      .height(350)
      .margins({ top: 10, right: 10, bottom: 40, left: 40 })
      .dimension(yearDim)
      .group(yearGroup)
      .x(xScale)
      .elasticY(true)
      .renderArea(false)
      .renderDataPoints(true)
      .clipPadding(10)
      .yAxisLabel("Número de alunos")
      .xAxisLabel("Ano")
      .brushOn(true);

    timeLineChart.render();
  });
</script>

<div bind:this={el}>
  <h3>{title}</h3>
</div>
