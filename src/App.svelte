<script>
  import { onMount } from "svelte";
  import * as d3 from "d3";

  const dataset = d3.json("").then((data) => data);

  console.log(dataset);
</script>

<main>
  {console.log(dataset)}
</main>

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
