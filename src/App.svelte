<script>
  import * as d3 from 'd3';

  import Main from './Main.svelte'

  export let datasets = []
  
  let promise = getData();
  async function getData() {
    let moonD = await d3.csv("data/manmade_materials_full.csv")
    let moonGeo = await d3.json("data/moon.geo.json")
    datasets = await [moonD, moonGeo]
  }
</script>

  <main>
    {#await promise then data} 
      <Main data={datasets} />
    {/await}
  </main>

<style>
	main {
    width: 100%;
    margin: 0 auto;
	}
</style>