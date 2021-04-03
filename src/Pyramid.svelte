<script>
  import { onMount } from "svelte";
  import * as d3 from "d3";
  import crossfilter from "crossfilter2";

  export let dataset;

  function colorTransform(c1, c2) {
    const color1 = c1.replace("#", "");
    const origHex = {
      r: color1.substring(0, 2),
      g: color1.substring(2, 4),
      b: color1.substring(4, 6),
    };
    const transVec = {
      r: c2.substring(0, 2),
      g: c2.substring(2, 4),
      b: c2.substring(4, 6),
    };
    const newHex = {};

    function transform(d, e) {
      let f = parseInt(d, 16) + parseInt(e, 16);
      if (f > 255) {
        f = 255;
      }
      return f.toString(16);
    }
    newHex.r = transform(origHex.r, transVec.r);
    newHex.g = transform(origHex.g, transVec.g);
    newHex.b = transform(origHex.b, transVec.b);
    return "#" + newHex.r + newHex.g + newHex.b;
  }

  function translation(x, y) {
    return "translate(" + x + "," + y + ")";
  }

  function prettyFormat(x) {
    return x.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",");
  }

  const facts = crossfilter(dataset);

  const pyramidDim = facts.dimension(
    (d) => d.NU_ANO_CENSO + "/" + d.NU_IDADE + "/" + d.TP_SEXO
  );

  const pyramidDimCount = pyramidDim.group().all();

  const data = [];

  pyramidDimCount.forEach((d) => {
    if (Number(d.key.split("/")[1]) < 40) {
      data.push({
        year: Number(d.key.split("/")[0]),
        age: Number(d.key.split("/")[1]),
        sex: d.key.split("/")[2] === "2" ? "M" : "F",
        value: d.value,
      });
    } else {
    }
  });

  console.log(data);

  const margin = { top: 50, right: 10, bottom: 20, left: 10, middle: 20 };

  let height = 600;
  const fullHeight = height;

  let width = 500;
  const fullWidth = width;

  const sectorWidth = width / 2 - margin.middle;
  const leftBegin = sectorWidth - margin.left;
  const rightBegin = width - margin.right - sectorWidth;

  width = width - (margin.left + margin.right);
  height = height - (margin.top + margin.bottom);

  const style = {
    leftBarColor: "#6c9dc6",
    rightBarColor: "#de5454",
    tooltipBG: "#fefefe",
    tooltipColor: "black",
  };

  const totalPopulation = d3.sum(data, (d) => d.value);

  const percentage = (d) => d / totalPopulation;

  const maxValue =
    Math.ceil(d3.max(data, (d) => percentage(d.value)) / 0.05) * 0.05;

  var xScale = d3
    .scaleLinear()
    .domain([0, maxValue])
    .range([0, sectorWidth - margin.middle])
    .nice();

  var xScaleLeft = d3
    .scaleLinear()
    .domain([0, maxValue])
    .range([sectorWidth, 0]);

  var xScaleRight = d3
    .scaleLinear()
    .domain([0, maxValue])
    .range([0, sectorWidth]);

  var yScale = d3
    .scaleBand()
    .domain(
      data.map((d) => {
        return d.age;
      })
    )
    .range([height, 0], 0.1);

  var yAxisLeft = d3
    .axisRight()
    .scale(yScale)
    .tickSize(4, 0)
    .tickPadding(margin.middle - 4);

  var yAxisRight = d3.axisLeft().scale(yScale).tickSize(4, 0).tickFormat("");

  var xAxisRight = d3.axisBottom().scale(xScale).tickFormat(d3.format(".0%"));

  var xAxisLeft = d3
    .axisBottom()
    // REVERSE THE X-AXIS SCALE ON THE LEFT SIDE BY REVERSING THE RANGE
    .scale(xScale.copy().range([leftBegin, 0]))
    .tickFormat(d3.format(".0%"));

  let el;

  onMount(() => {
    const styleSection = d3
      .select(el)
      .append("style")
      .text(
        "svg {max-width:100%} \
    .axis line,axis path {shape-rendering: crispEdges;fill: transparent;stroke: #555;} \
    .axis text {font-size: 11px;} \
    .bar {fill-opacity: 0.5;} \
    .bar.left {fill: " +
          style.leftBarColor +
          ";} \
    .bar.left:hover {fill: " +
          colorTransform(style.leftBarColor, "333333") +
          ";} \
    .bar.right {fill: " +
          style.rightBarColor +
          ";} \
    .bar.right:hover {fill: " +
          colorTransform(style.rightBarColor, "333333") +
          ";} \
    .tooltip {position: absolute;line-height: 1.1em;padding: 7px; margin: 3px;background: " +
          style.tooltipBG +
          "; color: " +
          style.tooltipColor +
          "; pointer-events: none;border-radius: 6px;}"
      );

    const region = d3
      .select(el)
      .append("svg")
      .attr("width", fullWidth)
      .attr("height", fullHeight);

    const legend = region.append("g").attr("class", "legend");

    legend
      .append("rect")
      .attr("class", "bar left")
      .attr("x", width / 2 - margin.middle * 3)
      .attr("y", 12)
      .attr("width", 12)
      .attr("height", 12);

    legend
      .append("text")
      .attr("fill", "#000")
      .attr("x", width / 2 - margin.middle * 2)
      .attr("y", 18)
      .attr("dy", "0.32em")
      .text("Homens");

    legend
      .append("rect")
      .attr("class", "bar right")
      .attr("x", width / 2 + margin.middle * 2)
      .attr("y", 12)
      .attr("width", 12)
      .attr("height", 12);

    legend
      .append("text")
      .attr("fill", "#000")
      .attr("x", width / 2 + margin.middle * 3)
      .attr("y", 18)
      .attr("dy", "0.32em")
      .text("Mulheres");

    const tooltipDiv = d3
      .select("body")
      .append("div")
      .attr("class", "tooltip")
      .style("opacity", 0);

    const pyramid = region
      .append("g")
      .attr("class", "inner-region")
      .attr("transform", translation(margin.left, margin.top));

    const leftBarGroup = pyramid
      .append("g")
      .attr("transform", translation(leftBegin, 0) + "scale(-1,1)");

    const rightBarGroup = pyramid
      .append("g")
      .attr("transform", translation(rightBegin, 0));

    pyramid
      .append("g")
      .attr("class", "axis y left")
      .attr("transform", translation(leftBegin, 0))
      .call(yAxisLeft)
      .selectAll("text")
      .style("text-anchor", "middle");

    pyramid
      .append("g")
      .attr("class", "axis y right")
      .attr("transform", translation(rightBegin, 0))
      .call(yAxisRight);

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
  });
</script>

<div bind:this={el} class="chart" />
