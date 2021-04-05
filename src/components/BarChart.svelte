<script>
  import { afterUpdate, onMount } from "svelte";
  import * as dc from "dc";
  import * as d3 from "d3";

  export let cursosDim;

  let title = "Cursos com maior número de evasão - UFPE";

  const cursoGroup = cursosDim.group();

  const createOrdinalColors = (bins) => {
    const scheme = new Array(bins);
    for (let i = 0; i < bins; i++) {
      scheme[i] = d3.interpolatePlasma(i / bins / 2 + 0.3);
    }
    return scheme;
  };

  let el;

  afterUpdate(() => {
    let rowChart = dc.rowChart(el);

    rowChart
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

    //rowChart.on("filtered", (_, type) => update());

    rowChart.render();
  });
</script>

<div bind:this={el} class="col-6">
  <h3>{title}</h3>
</div>
