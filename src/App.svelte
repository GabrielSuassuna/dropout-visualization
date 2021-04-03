<script>
  import { onMount } from "svelte";
  import * as d3 from "d3";

  const dataset = d3
    .json(
      "https://raw.githubusercontent.com/GabrielSuassuna/dropout-visualization/master/data/data.json?token=AIYBXOUMBRZ6DJ6PZMAAGC3AOFQCY#"
    )
    .then((data) => console.log(data));
</script>

<main />

<style>
  main {
    text-align: center;
    padding: 1em;
    max-width: 240px;
    margin: 0 auto;
  }

  h1 {
    color: #ff3e00;
    text-transform: uppercase;
    font-size: 4em;
    font-weight: 100;
  }

  @media (min-width: 640px) {
    main {
      max-width: none;
    }
  }
</style>
