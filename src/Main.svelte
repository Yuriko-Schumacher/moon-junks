<script>
  import * as d3 from 'd3';

  import Title from './Title.svelte'
  import Scrolly from './Scrolly.svelte'
  import ThreeDMoon from './ThreeDMoon.svelte'

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

<Title />
<Scrolly data={data} />
<br><br><br><br><br><br><br><br><br>
<ThreeDMoon data={data} />