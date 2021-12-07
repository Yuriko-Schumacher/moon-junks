<script>
  import * as d3 from 'd3';

  import Main from './Main.svelte'

  export let moonD = [];
  
  let promise = getData();
  async function getData() {
    moonD = await d3.csv("data/manmade_materials_full.csv")
  }
</script>

  <main>
    {#await promise then data} 
      <Main data={moonD} />
    {/await}
  </main>

<style>
	main {
    width: 100%;
    margin: 0 auto;
	}
</style>