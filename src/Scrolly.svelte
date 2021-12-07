<script>
  import { onMount } from 'svelte';
  import * as d3 from "d3"
  import Scroller from "@sveltejs/svelte-scroller";
  import { spring } from 'svelte/motion';

  export let data;
  console.log(data)

  let index, offset, progress;
  const width = 900;
  const height = 500;
  const margin = {
    t: 50, r: 0, b: 0, l: 50
  }
  let nearProjection = d3.geoOrthographic();
  let farProjection = d3.geoOrthographic().rotate([180, 0])

  let rasterMoon;
  let xScale, yScale;

  onMount(
    async() => {
    rasterMoon = d3.selectAll(".rasterMoon")
    rasterMoon.each(function(p, j) {
      const projection = j === 0 ? nearProjection : farProjection

      d3.select(this)
        .style("transform", `scale(0.65) translate(-250px, ${-(200 * j)}px)`)
      
      const canvas = d3.select(this)
        .select("canvas")
        .attr("width", width)
        .attr("height", height)
        .attr('id', `raster-${j}`);

      const context = canvas.node().getContext("2d");

      const image = new Image;
      image.onload = onload;
      image.src = "./image/moon_lowres.jpeg";

      function onload() {
      let dx = image.width / 2;
      let dy = image.height / 2;

          context.drawImage(image, 0, 0, dx, dy);

          let sourceData = context.getImageData(0, 0, dx, dy).data;
          let target = context.createImageData(width, height)
          let targetData = target.data;

          //this get it done in terms of laying the right tiles on the right projection!
          for (let y = 0, i = -1; y < height; ++y) {
            for (let x = 0; x < width; ++x) {
              let p = projection.invert([x, y]), λ = p[0], φ = p[1];
              if (λ > 180 || λ < -180 || φ > 90 || φ < -90) { i += 4; continue; }
              let q = ((90 - φ) / 180 * dy | 0) * dx + ((180 + λ) / 360 * dx | 0) << 2;
              targetData[++i] = sourceData[q];
              targetData[++i] = sourceData[++q];
              targetData[++i] = sourceData[++q];
              targetData[++i] = 255;
            }
          }
          context.clearRect(0, 0, width, height);
          context.putImageData(target, 0, 0);
      }
    })
  })

  const move = (cx, cy) => `transform: translate(${cx}px, ${cy}px)`;
  const colors = {
    yellow : {
      r: 255,
      g: 233,
      b: 0
    },
    purple : {
      r: 160,
      g: 32,
      b: 240
    },
    white : 238,
    black : 34
  }

  let circles = spring(
    data.map((d) => ({
      cx: 0,
      cy: 0,
      cr: 0,
      opacity: 1,
      r: 0,
      g: 0,
      b: 0
    })),
    {
      stiffness: 0.1,
      damping: 0.9
    }
  );

  let newCircles;

  let rasterOpacity = spring({ near: 1, far: 0},
    {
      stiffness: 0.1,
      damping: 0.9
    });
  let newRasterOpacity;

  $: {
    if (index === 0) {
      newRasterOpacity = {
        near: 1,
        far: 0
      }
      newCircles = data.map((d) => ({
        cx: Math.abs(d.east) < 90 ? nearProjection([d.east, d.north])[0] : farProjection([d.east, d.north])[0],
        cy: Math.abs(d.east) < 90 ? nearProjection([d.east, d.north])[1] : farProjection([d.east, d.north])[1] + 585,
        cr: 10,
        opacity: 0,
        r: Math.abs(d.east) < 90 ? colors.yellow.r : colors.purple.r,
        g: Math.abs(d.east) < 90 ? colors.yellow.g : colors.purple.g,
        b: Math.abs(d.east) < 90 ? colors.yellow.b : colors.purple.b,
      }))
    } else
    if (index === 1) {
      newRasterOpacity = {
        near: 0.3,
        far: 1
      }
      newCircles = data.map((d) => ({
        cx: Math.abs(d.east) < 90 ? nearProjection([d.east, d.north])[0] : farProjection([d.east, d.north])[0],
        cy: Math.abs(d.east) < 90 ? nearProjection([d.east, d.north])[1] : farProjection([d.east, d.north])[1] + 585,
        cr: 10,
        opacity: 0,
        r: Math.abs(d.east) < 90 ? colors.yellow.r : colors.purple.r,
        g: Math.abs(d.east) < 90 ? colors.yellow.g : colors.purple.g,
        b: Math.abs(d.east) < 90 ? colors.yellow.b : colors.purple.b,
      }))
    } else
    if (index === 2) {
      newRasterOpacity = {
        near: 0.3,
        far: 1
      }
      newCircles = data.map((d) => ({
        cx: Math.abs(d.east) < 90 ? nearProjection([d.east, d.north])[0] : farProjection([d.east, d.north])[0],
        cy: Math.abs(d.east) < 90 ? nearProjection([d.east, d.north])[1] : farProjection([d.east, d.north])[1] + 585,
        cr: 10,
        opacity: d.object === "Chang'e 4" ? 1 : 0,
        r: Math.abs(d.east) < 90 ? colors.yellow.r : colors.purple.r,
        g: Math.abs(d.east) < 90 ? colors.yellow.g : colors.purple.g,
        b: Math.abs(d.east) < 90 ? colors.yellow.b : colors.purple.b,
      }))
    } else 
    if (index === 3) {
      newRasterOpacity = {
        near: 1,
        far: 0.5
      }
      newCircles = data.map((d) => ({
        cx: Math.abs(d.east) < 90 ? nearProjection([d.east, d.north])[0] : farProjection([d.east, d.north])[0],
        cy: Math.abs(d.east) < 90 ? nearProjection([d.east, d.north])[1] : farProjection([d.east, d.north])[1] + 585,
        cr: 10,
        opacity: d.launch_year === 1959 ? 1 : 0,
        r: Math.abs(d.east) < 90 ? colors.yellow.r : colors.purple.r,
        g: Math.abs(d.east) < 90 ? colors.yellow.g : colors.purple.g,
        b: Math.abs(d.east) < 90 ? colors.yellow.b : colors.purple.b,
      }))
    } else 
    if (index === 4 || index === 6) {
      newRasterOpacity = {
        near: 1,
        far: 1
      }
      newCircles = data.map((d) => ({
        cx: Math.abs(d.east) < 90 ? nearProjection([d.east, d.north])[0] : farProjection([d.east, d.north])[0],
        cy: Math.abs(d.east) < 90 ? nearProjection([d.east, d.north])[1] : farProjection([d.east, d.north])[1] + 585,
        cr: 10,
        opacity: d.east === 0 || d.object === "Chang'e 4" ? 0 : 1,
        r: Math.abs(d.east) < 90 ? colors.yellow.r : colors.purple.r,
        g: Math.abs(d.east) < 90 ? colors.yellow.g : colors.purple.g,
        b: Math.abs(d.east) < 90 ? colors.yellow.b : colors.purple.b,
      }))
    } else 
    if (index === 5) {
      newRasterOpacity = {
        near: 1,
        far: 1
      }
      newCircles = data.map((d) => ({
        cx: Math.abs(d.east) < 90 ? nearProjection([d.east, d.north])[0] : farProjection([d.east, d.north])[0],
        cy: Math.abs(d.east) < 90 ? nearProjection([d.east, d.north])[1] : farProjection([d.east, d.north])[1] + 585,
        cr: 10,
        opacity: d.is_apollo === "TRUE" && d.east !== 0 ? 1 : 0,
        r: Math.abs(d.east) < 90 ? colors.yellow.r : colors.purple.r,
        g: Math.abs(d.east) < 90 ? colors.yellow.g : colors.purple.g,
        b: Math.abs(d.east) < 90 ? colors.yellow.b : colors.purple.b,
      }))
    } else 
    if (index === 7) {
      xScale = d3.scaleLinear().domain([1, 30]).range([margin.l, width - 300])
      yScale = d3.scaleLinear().domain([1958, 2009]).range([height * 2 - margin.b, margin.t])

      newRasterOpacity = {
        near: 0,
        far: 0
      }
      newCircles = data.map((d) => ({
        cx: d.by_year !== "NA" ? xScale(d.by_year % 50) : 0,
        cy: d.by_year !== "NA" ? yScale(d.launch_year) : 0,
        cr: 10,
        opacity: d.by_year === "NA" ? 0 : 1,
        r: Math.abs(d.east) < 90 ? colors.yellow.r : colors.purple.r,
        g: Math.abs(d.east) < 90 ? colors.yellow.g : colors.purple.g,
        b: Math.abs(d.east) < 90 ? colors.yellow.b : colors.purple.b,
      }))
    } else 
    if (index === 8) {
      xScale = d3.scaleLinear().domain([1, 30]).range([margin.l, width - 300])
      yScale = d3.scaleBand().domain(["US", "Soviet", "India", "China", "Japan", "ESA"]).range([height * 2 - margin.b, margin.t])

      newRasterOpacity = {
        near: 0,
        far: 0
      }
      newCircles = data.map((d) => ({
        cx: d.by_country !== "NA" ? xScale(d.by_country % 50) : 0,
        cy: d.by_country !== "NA" ? yScale(d.country) : 0,
        cr: 10,
        opacity: d.by_year === "NA" ? 0 : 1,
        r: Math.abs(d.east) < 90 ? colors.yellow.r : colors.purple.r,
        g: Math.abs(d.east) < 90 ? colors.yellow.g : colors.purple.g,
        b: Math.abs(d.east) < 90 ? colors.yellow.b : colors.purple.b,
      }))
    }
    rasterOpacity.set(newRasterOpacity)
    circles.set(newCircles)
  } 
</script>

<Scroller top="{0}" bottom="{1}" bind:index bind:offset bind:progress>
  <div class="background" slot="background">
    <div class="tooltip">
      <strong><span id="object">Object</span></strong
      ><br />
      <span id="country">Country</span>
      <span id="category">Category</span><br />
      <span id="year">Year</span>
    </div>
    <div class="rasterMoon" style="opacity: {$rasterOpacity.near}">
      <canvas></canvas>
      <svg width={width + 10} height={height + 10}>
        <defs>
          <mask id="moon">
              <circle id="outer" cx="480" cy="250" r="800" fill="white"/>
              <circle id="inner" cx="480" cy="250" r="250"/>
          </mask>
        </defs>
        <rect
          x="-5"
          y="-5"
          width="{width + 10}"
          height="{height + 10}"
          fill="rgb(0, 0, 0)"
          mask="url(#moon)"
        ></rect>
      </svg>
    </div>
    <div class="rasterMoon" style="opacity: {$rasterOpacity.far}">
      <canvas></canvas>
      <svg width={width + 10} height={height + 10}>
        <defs>
          <mask id="moon">
              <circle id="outer" cx="480" cy="250" r="800" fill="white"/>
              <circle id="inner" cx="480" cy="250" r="250"/>
          </mask>
        </defs>
        <rect
          x="-5"
          y="-5"
          width="{width + 10}"
          height="{height + 10}"
          fill="rgb(0, 0, 0)"
          mask="url(#moon)"
        ></rect>
      </svg>
    </div>
    <svg width={width} height={height * 2}>
      <g transform="scale(0.65) translate(210 135)">
        {#each $circles as {cx, cy, cr, opacity, r, g, b}} 
          <circle
            style="{move(cx, cy)}"
            r="{cr}"
            stroke="#222222"
            stroke-width="3"
            stroke-opacity="{opacity}"
            fill="rgb({r}, {g}, {b})"
            fill-opacity="{opacity}"
          />
        {/each}
      </g>
    </svg>
  </div>

  <div class="foreground" slot="foreground">
    <section class="step" data-section-id="0">
      <p>
        This side of the Moon may look familiar. This is the near side, and we can only see this side of the Moon.
      </p>
    </section>
    <section class="step" data-section-id="1">
      <p>
        And the image below shows the far side. From Earth, we’d need to go around the Moon to reach this side.
      </p>
    </section>
    <section class="step" data-section-id="2">
      <p>
        Chang’e 4’s landing site is spotted here. The rover Yutu 2, as well as the “mysterious hut” spotted by the rover is a few hundred meters away from the landing location.
      </p>
    </section>
    <section class="step" data-section-id="3">
      <p>
        The first human-made object to touch the Moon was in 1959, the Soviet Union's Luna 2.
      </p>
    </section>
    <section class="step" data-section-id="4">
      <p>
        In NASA’s catalogue of man-made objects, out of over 800 objects and landing points, 64 include available locations. 
      </p>
    </section>
    <section class="step" data-section-id="5">
      <p>
        Out of 741 objects without location informations, 724 were brought by Apollo missions in the late 1960s and early 1970s. Objects brought through Apollo missions and their landing locations are shown here.
      </p>
    </section>
    <section class="step" data-section-id="6">
      <p>
        Overall, there have been far less human activities on the far side. The hurdles are simply higher on far-side missions.
      </p>
    </section>
    <section class="step" data-section-id="7">
      <p>
        In the late 60s to early 70s, the Soviet Union and the U.S. were in the “Space Race” -- the competition during the Cold War to achieve superior spaceflight capability.
      </p>
      <p>
        The countries continuously conducted space missions, and a vast number of man-made objects were left on the Moon.
      </p>
      <p>
        There was a huge spike in the number of human artifacts on the moon in the late 1960s and 1970s.
      </p>
    </section>
    <section class="step" data-section-id="8">
      <p>
        More than 96% of those objects were left by the U.S., followed by the Soviet Union. Other countries include China, India, Japan, and the European Space Agency (ESA).
      </p>
    </section>
  </div>
</Scroller>

<style>

  .rasterMoon {
    position: relative;
  }

  svg {
    position: absolute;
    top: 0;
    left: 0;
  }

  .tooltip {
    display: none;
    position: fixed;
    top: 0;
    left: 0;
    background: rgba(0, 0, 0, 0.75);
    padding: 0.8em 1em;
    border-radius: 0.3em;
  }

  section {
    height: 80vh;
  }

  .background {
    /* background: rgba(255, 255, 255, 0.1); */
  }

  .foreground {
    margin: 0 0 0 auto;
    width: calc(100vw - 600px);
    max-width: 800px;
    /* background: rgba(64, 224, 208, 0.1); */
  }

  .step {
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
    padding: 8em;
  }
</style>