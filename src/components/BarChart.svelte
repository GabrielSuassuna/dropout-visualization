<script>
  import { onMount } from "svelte";
  import * as dc from "dc";
  import * as d3 from "d3";

  export let cursosDim;
  export let update;

  let title = "Cursos com maior número de evasão - UFPE";

  const width = 600;

  const cursoGroup = cursosDim.group();

  const createOrdinalColors = (bins) => {
    const scheme = new Array(bins);
    for (let i = bins; i >= 0; i--) {
      scheme[i] = d3.interpolateBlues(i / bins / 2 + 0.5);
    }
    return scheme;
  };

  let el;

  onMount(() => {
    let rowChart = dc.rowChart(el);

    rowChart
      .width(width - 100)
      .height(400)
      .dimension(cursosDim)
      .group(cursoGroup)
      .ordinalColors(createOrdinalColors(10))
      .elasticX(true)
      .ordering(function (d) {
        return -d.value;
      })
      .othersGrouper(false)
      .cap(10);

    rowChart.on("filtered", (_, type) => update());

    rowChart.render();
  });
</script>

<div bind:this={el} class="chart">
  <h3>{title}</h3>
</div>
