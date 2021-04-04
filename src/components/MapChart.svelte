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

<style>
  #mapid {
    width: 750px;
    height: 590px;
  }
  .info {
    padding: 6px 8px;
    font: 14px/16px Arial, Helvetica, sans-serif;
    background: white;
    background: rgba(255, 255, 255, 0.8);
    box-shadow: 0 0 15px rgba(0, 0, 0, 0.2);
    border-radius: 5px;
  }
  .info h4 {
    margin: 0 0 5px;
    color: #777;
  }

  .legend {
    text-align: left;
    line-height: 18px;
    color: #555;
  }
  .legend i {
    width: 18px;
    height: 18px;
    float: left;
    margin-right: 8px;
    opacity: 0.7;
  }
</style>
