<script>
  import * as d3 from 'd3';

  import Intro from './Intro.svelte'
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

<Intro />
<br><br><br><br><br><br><br><br><br>
<Scrolly data={data} />
<br><br><br><br><br><br><br><br><br>
<ThreeDMoon data={data} />