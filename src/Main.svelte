<script>
  import * as d3 from 'd3';

  import Intro from './Intro.svelte'
  import Scrolly from './Scrolly.svelte'
  import ThreeDMoon from './ThreeDMoon.svelte'
  import Footer from "./Footer.svelte"

  export let data;

  let moonD = data[0]
  let moonGeo = data[1]

  const parseData = (data) => {
    data.map(d => {
      d.by_year = +d.by_year;
      d.by_country = +d.by_country;
      d.launch_year = +d.launch_year;
      d.north = +d.north;
      d.east = +d.east;
      d.id = +d.id;
    })
  }
  parseData(moonD)
</script>

<Intro />
<br><br><br><br><br><br><br><br><br>
<Scrolly data={moonD} geo={moonGeo}/>
<section>
  <article>
    <h2>
      None of our business?
    </h2>
    <p>
      The number of human-made objects, or moon junk, is only increasing.
    </p>
    <p>
      They might be small compared to the whole moon, or they might be so far away from us. But think about the Moon you might see tonight.
    </p>
    <p>
      How many human-made objects would be on the visible areas of the Moon from where you are?
    </p>
    <p>
      Protecting the Earth may sound our priority, but it doesn't mean we are encouraged to move pollutants to space.
    </p>
    <p>
      <a href="https://yuriko-schumacher.github.io/moon-three/">Explore human-made objects on the moon</a>
    </p>
  </article>
</section>
<!-- <ThreeDMoon data={moonD} /> -->
<Footer />