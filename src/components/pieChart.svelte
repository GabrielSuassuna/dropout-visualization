<script>
  import { onMount } from "svelte";
  import * as dc from "dc";

  export let raceDim;

  let title = "Taxas de evasão por etnia - UFPE";

  const raceGroup = raceDim.group();

  let el;

  onMount(() => {
    let pieChart = dc.pieChart(el);

    pieChart
      .height(480)
      .slicesCap(4)
      .innerRadius(100)
      .dimension(raceDim)
      .group(raceGroup)
      .legend(dc.legend().highlightSelected(true))
      // workaround for #703: not enough data is accessible through .label() to display percentages
      .on("pretransition", function (chart) {
        chart.selectAll("text.pie-slice").text(function (d) {
          return (
            d.data.key +
            " " +
            dc.utils.printSingleValue(
              ((d.endAngle - d.startAngle) / (2 * Math.PI)) * 100
            ) +
            "%"
          );
        });
      });

    pieChart.render();
  });
</script>

<div bind:this={el} class="col-6">
  <h3>{title}</h3>
</div>
