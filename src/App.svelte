<script>
  import * as d3 from "d3";
  import BarChart from "./components/BarChart.svelte";
  import Pyramid from "./components/PyramidChart.svelte";
  import crossfilter from "crossfilter2";
  import LineChart from "./components/LineChart.svelte";
  import MapChart from "./components/MapChart.svelte";
  import { afterUpdate, beforeUpdate, onMount } from "svelte";
  import PieChart from "./components/pieChart.svelte";

  let cursosDim;
  let yearDim;
  let sitDim;
  let ufDim;
  let pyramidDim;
  let geojson;
  let raceDim;
  let pyramidData = [];

  async function getData() {
    const data = await d3.csv(
      "https://raw.githubusercontent.com/GabrielSuassuna/dropout-visualization/master/data/data.csv?token=AIYBXOUHMZUV3ANMLGBMCD3AOFRTQ"
    );

    const changeRace = (type) => {
      switch (type) {
        case "0":
          return "Não Declarado";
        case "1":
          return "Branca";
        case "2":
          return "Preta";
        case "3":
          return "Parda";
        case "4":
          return "Amarela";
        case "5":
          return "Indigena";
        case "9":
          return "Não Declarado";
      }
    };

    let parseDate = d3.utcParse("%Y");

    data.forEach((d) => {
      d.TP_COR_RACA = changeRace(d.TP_COR_RACA);
      d.NU_ANO_CENSO = parseDate(d.NU_ANO_CENSO);
    });

    geojson = await d3.json(
      "https://raw.githubusercontent.com/luamzi/Data-Visualization/main/data/brazil_geo.json"
    );

    const facts = crossfilter(data);
    cursosDim = facts.dimension((d) => d.NO_CURSO);
    yearDim = facts.dimension((d) => d.NU_ANO_CENSO);
    sitDim = facts.dimension((d) => d.TP_SITUACAO);
    ufDim = facts.dimension((d) => d.UF);
    pyramidDim = facts.dimension((d) => [d.NU_IDADE, d.TP_SEXO]);
    raceDim = facts.dimension((d) => d.TP_COR_RACA);

    getPyramidData();

    sitDim.filter("4");
  }

  const getPyramidData = () => {
    pyramidData = [];
    const pyramidDimCount = pyramidDim.group().all();

    let plus40Male = {
      age: "40+",
      sex: "M",
      value: 0,
    };

    let plus40Female = {
      age: "40+",
      sex: "F",
      value: 0,
    };

    pyramidDimCount.forEach((d) => {
      if (Number(d.key[0]) < 40) {
        pyramidData.push({
          age: Number(d.key[0]),
          sex: d.key[1] === "2" ? "M" : "F",
          value: d.value,
        });
      } else {
        if (d.key[1] === "2") {
          plus40Male.value += d.value;
        } else {
          plus40Female.value += d.value;
        }
      }
    });

    pyramidData.push(plus40Female);
    pyramidData.push(plus40Male);
  };

  let promise = getData();
</script>

<main>
  {#await promise}
    <div class="container loading">
      <div class="row align-items-center justify-content-md-center">
        <h2>Carregando...</h2>
      </div>
    </div>
  {:then data}
    <header>
      <nav class="navbar fixed-top navbar-light bg-light">
        <a class="navbar-brand" href="#">
          <img
            src="https://upload.wikimedia.org/wikipedia/commons/thumb/8/85/Bras%C3%A3o_da_UFPE.png/800px-Bras%C3%A3o_da_UFPE.png"
            height="50"
            alt="logo-ufpe"
          />
          Análise de evasão acadêmica da UFPE
        </a>
      </nav>
    </header>
    <div class="container">
      <div class="row">
        <MapChart {ufDim} geoJsonData={geojson} />
        <Pyramid data={pyramidData} />
      </div>
      <LineChart {yearDim} {sitDim} />
      <div class="row">
        <BarChart {cursosDim} />
        <PieChart {raceDim} />
      </div>
    </div>
  {:catch error}
    <p style="color: red">{error.message}</p>
  {/await}
</main>

<style>
  main {
    padding: 1em;
    max-width: 240px;
    margin: 0 auto;
  }

  .container {
    margin-top: 60px;
  }

  .loading {
    height: 100%;
  }

  @media (min-width: 640px) {
    main {
      max-width: none;
    }
  }
</style>
