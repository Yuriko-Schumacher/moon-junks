<script>
  import * as d3 from 'd3';

  import Intro from './Intro.svelte'
  import Scrolly from './Scrolly.svelte'
  import ThreeDMoon from './ThreeDMoon.svelte'
  import Footer from "./Footer.svelte"

  export let data;

  const parseData = (data) => {
    data.map(d => {
      d.by_year = +d.by_year;
      d.by_country = +d.by_country;
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
<section>
  <article>
    <h2>
      Non of our business?
    </h2>
    <p>
      The number of man-made objects, or moon junks are only increasing.
    </p>
    <p>
      They might be small compared to the whole moon, or they might be so far away from us. But think about the Moon you might see tonight.
    </p>
    <p>
      The Moon phase tonight will be Waxing Crescent, and within the area that is visible to us, there are 7 moon junks or human landing locations. 
    </p>
    <p>
      Protecting the Earth may sound our priority, but it doesn't mean we are encouraged to move pollutants to the space.
    </p>
  </article>
</section>
<br><br><br><br><br><br><br><br><br>
<ThreeDMoon data={data} />
<Footer />