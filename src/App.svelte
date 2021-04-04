<script>
  import * as d3 from "d3";
  import BarChart from "./components/BarChart.svelte";
  import Pyramid from "./components/PyramidChart.svelte";
  import crossfilter from "crossfilter2";
  import LineChart from "./components/LineChart.svelte";
  import MapChart from "./components/MapChart.svelte";
  import { afterUpdate, beforeUpdate, onMount } from "svelte";

  let cursosDim;
  let yearDim;
  let sitDim;
  let ufDim;
  let pyramidDim;
  let geojson;
  let scatterDim;
  let pyramidData = [];

  async function getData() {
    const data = await d3.csv(
      "https://raw.githubusercontent.com/GabrielSuassuna/dropout-visualization/master/data/data.csv?token=AIYBXOUHMZUV3ANMLGBMCD3AOFRTQ"
    );

    data.forEach((d) => {
      d["Formado"] = d.TP_SITUACAO === "6" ? 1 : 0;
      d["Desvinculado"] = d.TP_SITUACAO === "4" ? 1 : 0;
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
    scatterDim = facts.dimension((d) => [d.NO_CURSO, d.TP_SITUACAO]);

    console.log(scatterDim.group().all());
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
    console.log(pyramidData);
    console.log(2);
  };

  const updatePyramidData = () => {
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

    const totalPopulation = d3.sum(pyramidData, (d) => d.value);

    const percentage = (d) => d / totalPopulation;

    const maxValue =
      Math.ceil(d3.max(pyramidData, (d) => percentage(d.value)) * 100) / 100;

    const xScale = d3
      .scaleLinear()
      .domain([0, maxValue])
      .range([0, sectorWidth - margin.middle])
      .nice();

    const yScale = d3
      .scaleBand()
      .domain(
        pyramidData.map((d) => {
          return d.age;
        })
      )
      .range([height, 0], 0.1);

    const yAxisLeft = d3
      .axisRight()
      .scale(yScale)
      .tickSize(4, 0)
      .tickPadding(margin.middle - 4);

    const yAxisRight = d3
      .axisLeft()
      .scale(yScale)
      .tickSize(4, 0)
      .tickFormat("");

    const xAxisRight = d3
      .axisBottom()
      .scale(xScale)
      .tickFormat(d3.format(".0%"));

    const xAxisLeft = d3
      .axisBottom()
      // REVERSE THE X-AXIS SCALE ON THE LEFT SIDE BY REVERSING THE RANGE
      .scale(xScale.copy().range([leftBegin, 0]))
      .tickFormat(d3.format(".0%"));

    const svg = d3.select("body").transition();

    svg.select(".axis y left").duration(750).call(yAxisLeft);

    svg.select("axis y right").duration(750).call(yAxisRight);

    pyramid
      .append("g")
      .attr("class", "axis x left")
      .attr("transform", translation(0, height))
      .call(xAxisLeft);

    pyramid
      .append("g")
      .attr("class", "axis x right")
      .attr("transform", translation(rightBegin, height))
      .call(xAxisRight);

    leftBarGroup
      .selectAll(".bar.left")
      .data(data.filter((d) => d.sex === "M"))
      .enter()
      .append("rect")
      .attr("class", "bar left")
      .attr("x", 0)
      .attr("y", (d) => yScale(d.age) + margin.middle / 4)
      .attr("width", (d) => xScale(percentage(d.value)))
      .attr(
        "height",
        Math.abs(yScale.range()[0] / data.length - margin.middle * 1.1)
      )
      .on("mouseover", function (event, d) {
        tooltipDiv.transition().duration(200).style("opacity", 0.9);
        tooltipDiv
          .html(
            "<strong>Idade Homens " +
              d.age +
              "</strong>" +
              "<br />  Alunos: " +
              prettyFormat(d.value) +
              "<br />" +
              Math.round(percentage(d.value) * 1000) / 10 +
              "% do Total"
          )
          .style("left", event.pageX + "px")
          .style("top", event.pageY - 28 + "px");
      })
      .on("mouseout", function (d) {
        tooltipDiv.transition().duration(500).style("opacity", 0);
      });

    rightBarGroup
      .selectAll(".bar.right")
      .data(data.filter((d) => d.sex === "F"))
      .enter()
      .append("rect")
      .attr("class", "bar right")
      .attr("x", 0)
      .attr("y", (d) => {
        return yScale(d.age) + margin.middle / 4;
      })
      .attr("width", (d) => {
        return xScale(percentage(d.value));
      })
      .attr(
        "height",
        Math.abs(yScale.range()[0] / data.length - margin.middle * 1.1)
      )
      .on("mouseover", (event, d) => {
        tooltipDiv.transition().duration(200).style("opacity", 0.9);
        tooltipDiv
          .html(
            "<strong> Idade Mulheres " +
              d.age +
              "</strong>" +
              "<br />  Alunos: " +
              prettyFormat(d.value) +
              "<br />" +
              Math.round(percentage(d.value) * 1000) / 10 +
              "% do Total"
          )
          .style("left", event.pageX + "px")
          .style("top", event.pageY - 28 + "px");
      })
      .on("mouseout", (d) => {
        tooltipDiv.transition().duration(500).style("opacity", 0);
      });

    svg = d3.select("body").transition();

    // Make the changes
    svg
      .select(".rect") // change the line
      .duration(750)
      .attr("d", valueline(data));
    svg
      .select(".x.axis") // change the x axis
      .duration(750)
      .call(xAxis);
    svg
      .select(".y.axis") // change the y axis
      .duration(750)
      .call(yAxis);
  };

  let promise = getData();
</script>

<main>
  {#await promise}
    <p>...waiting</p>
  {:then data}
    <Pyramid data={pyramidData} />
    <BarChart
      {cursosDim}
      update={(type) => {
        console.log(1);
        updatePyramidData(pyramidDim);
      }}
    />
    <LineChart {yearDim} {sitDim} />
    <MapChart {ufDim} geoJsonData={geojson} />
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
