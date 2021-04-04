<script>
  import { onMount } from "svelte";
  import * as dc from "dc";
  import * as d3 from "d3";
  import crossfilter from "crossfilter2";
  import * as L from "leaflet";

  export let facts;
  export let geoJsonData;

  let title = "Taxas de evasão por ano - UFPE";

  const UFDim = facts.dimension((d) => d.UF);

  const evasionByUFGroup = UFDim.group();

  console.log(evasionByUFGroup.all());

  const colorScheme = d3.schemeYlOrBr[7];

  const colorScale = d3.scaleQuantize().domain([0, 426]).range(colorScheme);

  let evasionMap = new Map();
  evasionByUFGroup.all().forEach((d) => evasionMap.set(d.key, d.value));

  onMount(() => {
    let map = L.map("mapid").setView([-15.5002199999999, -48.036034], 4);
    L.tileLayer(
      "https://cartodb-basemaps-{s}.global.ssl.fastly.net/light_all/{z}/{x}/{y}.png",
      {
        attribution: `&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>,
 Map tiles by &copy; <a href="https://carto.com/attribution">CARTO</a>`,
        maxZoom: 18,
      }
    ).addTo(map);

    function highlightFeature(e) {
      let layer = e.target;
      //console.log(e.target)

      layer.setStyle({
        weight: 2,
        color: "#AAA",
        dashArray: "",
        fillOpacity: 0.7,
      });

      if (!L.Browser.ie && !L.Browser.opera) {
        layer.bringToFront();
      }

      info.update(layer.feature);
    }

    let geojson;

    function resetHighlight(e) {
      geojson.resetStyle(e.target);
      info.update();
    }

    function zoomToFeature(e) {
      map.fitBounds(e.target.getBounds());
    }

    function onEachFeature(feature, layer) {
      layer.on({
        mouseover: highlightFeature,
        mouseout: resetHighlight,
        click: zoomToFeature,
      });
    }

    function style(feature) {
      return {
        weight: 1,
        opacity: 1,
        color: "white",
        dashArray: "3",
        fillOpacity: 0.6,
        fillColor: colorScale(evasionMap.get(feature.id)),
      };
    }

    geojson = L.geoJson(geoJsonData, {
      style: style,
      onEachFeature: onEachFeature,
    }).addTo(map);

    let legend = L.control({ position: "bottomright" });

    legend.onAdd = function (map) {
      let div = L.DomUtil.create("div", "info legend"),
        labels = [],
        n = colorScheme.length,
        from,
        to;

      for (let i = 0; i < n; i++) {
        let c = colorScheme[i];
        let fromto = colorScale.invertExtent(c);
        labels.push(
          '<i style="background:' +
            colorScheme[i] +
            '"></i> ' +
            d3.format("d")(fromto[0]) +
            (d3.format("d")(fromto[1])
              ? "&ndash;" + d3.format("d")(fromto[1])
              : "+")
        );
      }

      div.innerHTML = labels.join("<br>");
      return div;
    };

    legend.addTo(map);

    let info = L.control();

    info.onAdd = function (map) {
      this._div = L.DomUtil.create("div", "info");
      this.update();
      return this._div;
    };

    info.update = function (feat) {
      this._div.innerHTML =
        "<h5>Número de homicídios</h5>" +
        (feat
          ? "<b>" +
            feat.properties.NOME +
            "</b><br />" +
            evasionMap.get(feat.properties.UF) +
            " homicídios/100000 hab."
          : "Passe o mouse sobre um bairro");
    };

    info.addTo(map);
  });
</script>

<div role="main" class="container">
  <div class="row">
    <h3>Homicídios em Fortaleza em 2012</h3>
  </div>
  <div id="mapid" class="row" />
  <p>
    Dados retirados do site da <a href="http://www.sspds.ce.gov.br/">SSPDS</a>
  </p>
</div>

<style>
  #mapid {
    width: 740px;
    height: 590px;
  }
  :global(.info) {
    padding: 6px 8px;
    font: 14px/16px Arial, Helvetica, sans-serif;
    background: white;
    background: rgba(255, 255, 255, 0.8);
    box-shadow: 0 0 15px rgba(0, 0, 0, 0.2);
    border-radius: 5px;
  }
  :global(.info h4) {
    margin: 0 0 5px;
    color: #777;
  }

  :global(.legend) {
    text-align: left;
    line-height: 18px;
    color: #555;
  }
  :global(.legend i) {
    width: 18px;
    height: 18px;
    float: left;
    margin-right: 8px;
    opacity: 0.7;
  }
</style>
