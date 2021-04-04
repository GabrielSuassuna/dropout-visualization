<script>
  import * as d3 from "d3";
  import BarChart from "./components/BarChart.svelte";
  import Pyramid from "./components/PyramidChart.svelte";
  import crossfilter from "crossfilter2";
  import LineChart from "./components/LineChart.svelte";
  import MapChart from "./components/MapChart.svelte";

  async function getData() {
    const data = await d3.csv(
      "https://raw.githubusercontent.com/GabrielSuassuna/dropout-visualization/master/data/data.csv?token=AIYBXOUHMZUV3ANMLGBMCD3AOFRTQ"
    );

    return crossfilter(data);
  }
  let promise = getData();
</script>

<main>
  {#await promise}
    <p>...waiting</p>
  {:then facts}
    <Pyramid {facts} />
    <BarChart {facts} />
    <LineChart {facts} />
    <MapChart {facts} />
  {:catch error}
    <p style="color: red">{error.message}</p>
  {/await}
</main>

<style>
  main {
    text-align: center;
    padding: 1em;
    max-width: 240px;
    margin: 0 auto;
  }

  @media (min-width: 640px) {
    main {
      max-width: none;
    }
  }
</style>
