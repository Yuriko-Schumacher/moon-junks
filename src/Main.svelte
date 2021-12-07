<script>
  import * as d3 from 'd3';

  import TwoDMoon from './TwoDMoon.svelte'
  // import ThreeDMoon from './ThreeDMoon.svelte'

  export let data;

  const parseData = (data) => {
    data.map(d => {
      d.launch_year = +d.launch_year;
      d.north = +d.north;
      d.east = +d.east;
    })
  }
  parseData(data)
</script>

<TwoDMoon data={data} />
<!-- <ThreeDMoon data={data} /> -->