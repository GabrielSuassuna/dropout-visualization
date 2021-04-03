<script>
  import { onMount } from "svelte";
  import * as dc from "dc";
  import crossfilter from "crossfilter2";

  export let dataset;

  let title = "Cursos com maior número de evasão - UFPE";

  const width = 600;

  const colors = [
    "#6e40aa",
    "#6450c2",
    "#5564d5",
    "#447adf",
    "#3292e1",
    "#23abd8",
    "#1ac3c7",
    "#1ad7af",
    "#24e793",
    "#39f279",
  ];

  const facts = crossfilter(dataset);

  const cursosDim = facts.dimension((d) => d.NO_CURSO);

  const cursoGroup = cursosDim.group();

  console.log(cursoGroup.all());

  let el;

  onMount(() => {
    let rowChart = dc.rowChart(el);

    rowChart
      .width(width - 100)
      .height(400)
      .dimension(cursosDim)
      .group(cursoGroup, "Curso")
      .ordinalColors(colors)
      .elasticX(true)
      .ordering(function (d) {
        return -d.value;
      })
      .othersGrouper(false)
      .cap(10);

    rowChart.render();
  });
</script>

<div bind:this={el} class="chart">
  <h3>{title}</h3>
</div>
