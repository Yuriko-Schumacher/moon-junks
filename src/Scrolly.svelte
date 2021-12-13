<script>
  import { afterUpdate, onMount } from 'svelte';
  import * as d3 from "d3"
  import Scroller from "@sveltejs/svelte-scroller";
  import { spring } from 'svelte/motion';

  export let data;
  export let geo;

  let index, offset, progress;
  const width = window.innerWidth / 2;
  const height = window.innerHeight / 2;
  const margin = {
    t: 75, r: 25, b: 25, l: 25
  }
  const innerWidth = width - margin.l - margin.r > 520 ? width - margin.l - margin.r : 520;
  const innerHeight = height - margin.t - margin.b;
  const moonSides = [
    {
      side: "near",
      projection: d3.geoOrthographic().fitSize([innerWidth, innerHeight], geo),
    },
    {
      side: "far",
      projection: d3.geoOrthographic().rotate([180, 0]).fitSize([innerWidth, innerHeight], geo),
    }
  ]

  let rasterMoon;
  let xScale, yScale;

  onMount(
    async() => {

    // ----- DRAW RASTER MOON PROJECTION -----
    rasterMoon = d3.selectAll(".rasterMoon")
    rasterMoon.each(function(p, j) {
      const projection = moonSides[j].projection
      const canvas = d3.select(this).select("canvas")
      const context = canvas.node().getContext("2d");
      const canvasX = margin.l;
      const canvasY = margin.t;

      const image = new Image;
      image.onload = onload;
      image.src = "./image/moon_lowres.jpeg";

      function onload() {
      let dx = image.width / 2;
      let dy = image.height / 2;

          context.drawImage(image, canvasX, canvasY, dx, dy);

          let sourceData = context.getImageData(canvasX, canvasY, dx, dy).data;
          let target = context.createImageData(innerWidth, innerHeight)
          let targetData = target.data;

          //this get it done in terms of laying the right tiles on the right projection!
          for (let y = 0, i = -1; y < innerHeight; ++y) {
            for (let x = 0; x < innerWidth; ++x) {
              let p = projection.invert([x, y]), λ = p[0], φ = p[1];
              if (λ > 180 || λ < -180 || φ > 90 || φ < -90) { i += 4; continue; }
              let q = ((90 - φ) / 180 * dy | 0) * dx + ((180 + λ) / 360 * dx | 0) << 2;
              targetData[++i] = sourceData[q];
              targetData[++i] = sourceData[++q];
              targetData[++i] = sourceData[++q];
              targetData[++i] = 255;
            }
          }
          context.clearRect(canvasX, canvasY, innerWidth, innerHeight);
          context.putImageData(target, canvasX, canvasY);
      }
    })
  })

  afterUpdate(() => {
    // ----- TOOLTIP -----
    const backgroundContainer = d3.select(".scrolly__2d-moon").select(".background-container")
    backgroundContainer.style("z-index", "999").style("pointer-events", "auto")
    const circles = d3.select(".scrolly-circles").selectAll("circle")
    const tooltip = d3.select(".tooltip__2d")
    circles.on("mouseover", function(e) {
      let id = d3.select(this).attr("id");
      let thisD = data.filter(d => d.id === +id)[0]
      d3.select(this).attr('stroke-width', "5")
      tooltip
        .style("display", "block")
        .style("top", `${e.clientY + 5}px`)
        .style("left", `${e.clientX + 5}px`)
        .html(`<strong>${thisD.object}</strong><br>
                ${thisD.country} ${thisD.us_category} <br>
                ${thisD.launch_year}`)
    })
    circles.on("mouseout", function() {
      d3.select(this).attr("stroke-width", "1")
      tooltip
        .style("display", "none")
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
      r: 255,
      g: 0,
      b: 128
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
      b: 0,
      id: 0,
    })),
    {
      stiffness: 0.1,
      damping: 0.9
    }
  );

  let newCircles;

  let rasterOpacity = spring({ near: 0.8, far: 0},
    {
      stiffness: 0.1,
      damping: 0.9
    });
  let newRasterOpacity;

  let waffleOpacity;
  let waffleText = "";

  const years = [...Array(52).keys()].map(d => d + 1959)
  const countries = ["US", "Soviet", "Japan", "India", "China", "ESA"];

  $: {
    if (index === 0) {
      newRasterOpacity = {
        near: 0.8,
        far: 0
      }
      newCircles = data.map((d) => ({
        cx: d.north === 0 ? 0 : Math.abs(d.east) < 90 ? moonSides[0].projection([d.east, d.north])[0] + margin.l : moonSides[1].projection([d.east, d.north])[0] + margin.l,
        cy: d.north === 0 ? 0 : Math.abs(d.east) < 90 ? moonSides[0].projection([d.east, d.north])[1] + margin.t : moonSides[1].projection([d.east, d.north])[1] + height + margin.t,
        cr: innerWidth / 130,
        opacity: 0,
        r: colors.yellow.r,
        g: colors.yellow.g,
        b: colors.yellow.b,
        id: d.id,
      }))
      waffleOpacity = 0;
      waffleText = "";
    } else
    if (index === 1) {
      newRasterOpacity = {
        near: 0.3,
        far: 0.8
      }
      newCircles = data.map((d) => ({
        cx: d.north === 0 ? 0 : Math.abs(d.east) < 90 ? moonSides[0].projection([d.east, d.north])[0] + margin.l : moonSides[1].projection([d.east, d.north])[0] + margin.l,
        cy: d.north === 0 ? 0 : Math.abs(d.east) < 90 ? moonSides[0].projection([d.east, d.north])[1] + margin.t : moonSides[1].projection([d.east, d.north])[1] + height + margin.t,
        cr: innerWidth / 130,
        opacity: 0,
        r: colors.yellow.r,
        g: colors.yellow.g,
        b: colors.yellow.b,
        id: d.id,
      }))
    } else
    if (index === 2) {
      newRasterOpacity = {
        near: 0.3,
        far: 0.8
      }
      newCircles = data.map((d) => ({
        cx: d.north === 0 ? 0 : Math.abs(d.east) < 90 ? moonSides[0].projection([d.east, d.north])[0] + margin.l : moonSides[1].projection([d.east, d.north])[0] + margin.l,
        cy: d.north === 0 ? 0 : Math.abs(d.east) < 90 ? moonSides[0].projection([d.east, d.north])[1] + margin.t : moonSides[1].projection([d.east, d.north])[1] + height + margin.t,
        cr: innerWidth / 130,
        opacity: d.object === "Chang'e 4" ? 1 : 0,
        r: colors.yellow.r,
        g: colors.yellow.g,
        b: colors.yellow.b,
        id: d.id,
      }))
    } else 
    if (index === 3) {
      newRasterOpacity = {
        near: 0.8,
        far: 0.3
      }
      newCircles = data.map((d) => ({
        cx: d.north === 0 ? 0 : Math.abs(d.east) < 90 ? moonSides[0].projection([d.east, d.north])[0] + margin.l : moonSides[1].projection([d.east, d.north])[0] + margin.l,
        cy: d.north === 0 ? 0 : Math.abs(d.east) < 90 ? moonSides[0].projection([d.east, d.north])[1] + margin.t : moonSides[1].projection([d.east, d.north])[1] + height + margin.t,
        cr: innerWidth / 130,
        opacity: d.launch_year === 1959 ? 1 : 0,
        r: colors.yellow.r,
        g: colors.yellow.g,
        b: colors.yellow.b,
        id: d.id,
      }))
    } else 
    if (index === 4 || index === 6) {
      newRasterOpacity = {
        near: 0.8,
        far: 0.8
      }
      newCircles = data.map((d) => ({
        cx: d.north === 0 ? 0 : Math.abs(d.east) < 90 ? moonSides[0].projection([d.east, d.north])[0] + margin.l : moonSides[1].projection([d.east, d.north])[0] + margin.l,
        cy: d.north === 0 ? 0 : Math.abs(d.east) < 90 ? moonSides[0].projection([d.east, d.north])[1] + margin.t : moonSides[1].projection([d.east, d.north])[1] + height + margin.t,
        cr: innerWidth / 130,
        opacity: d.north === 0 || d.object === "Chang'e 4" ? 0 : 0.8,
        r: colors.yellow.r,
        g: colors.yellow.g,
        b: colors.yellow.b,
        id: d.id,
      }))
      waffleOpacity = 0;
      waffleText = "";
    } else 
    if (index === 5) {
      newRasterOpacity = {
        near: 0.8,
        far: 0.8
      }
      newCircles = data.map((d) => ({
        cx: d.north === 0 ? 0 : Math.abs(d.east) < 90 ? moonSides[0].projection([d.east, d.north])[0] + margin.l : moonSides[1].projection([d.east, d.north])[0] + margin.l,
        cy: d.north === 0 ? 0 : Math.abs(d.east) < 90 ? moonSides[0].projection([d.east, d.north])[1] + margin.t : moonSides[1].projection([d.east, d.north])[1] + height + margin.t,
        cr: innerWidth / 130,
        opacity: d.is_apollo === "TRUE" && d.north !== 0 ? 0.8 : 0,
        r: colors.yellow.r,
        g: colors.yellow.g,
        b: colors.yellow.b,
        id: d.id,
      }))
    } else 
    if (index === 7) {
      xScale = d3.scaleLinear().domain([0, 100]).range([150, width - margin.r])
      yScale = d3.scaleBand().domain(years).range([margin.t + 50, height * 2 - 100])

      newRasterOpacity = {
        near: 0,
        far: 0
      }
      newCircles = data.map((d) => ({
        cx: d.by_year !== "NA" ? xScale((d.by_year - 1) % 100) : 0,
        cy: d.by_year !== "NA" ? yScale(d.launch_year) + (Math.floor((d.by_year - 1) / 100) + 1) * innerWidth / 350 * 2 : 0,
        cr: innerWidth / 350,
        opacity: d.by_year === "NA" ? 0 : 1,
        r: colors.yellow.r,
        g: colors.yellow.g,
        b: colors.yellow.b,
        id: d.id,
      }))
      waffleOpacity = 1;
      waffleText = "Objects landed on the Moon by year";
    } else 
    if (index === 8) {
      xScale = d3.scaleLinear().domain([0, 100]).range([150, width - margin.r])
      yScale = d3.scaleBand().domain(countries).range([margin.t + 50, height * 2 - 100])

      newRasterOpacity = {
        near: 0,
        far: 0
      }
      newCircles = data.map((d) => ({
        cx: d.by_country !== "NA" ? xScale((d.by_country - 1) % 100) : 0,
        cy: d.by_country !== "NA" ? yScale(d.country) + (Math.floor((d.by_country - 1) / 100) + 1) * innerWidth / 350 * 2 : 0,
        cr: innerWidth / 350,
        opacity: d.by_year === "NA" ? 0 : 1,
        r: colors.yellow.r,
        g: colors.yellow.g,
        b: colors.yellow.b,
        id: d.id,
      }))
      waffleText = "Objects landed on the Moon by country";
    }
    rasterOpacity.set(newRasterOpacity)
    circles.set(newCircles)
  } 
</script>

<div class="scrolly__2d-moon">
  <Scroller top="{0}" bottom="{1}" bind:index bind:offset bind:progress>
    <div class="background" slot="background">
      <div class="tooltip tooltip__2d">
        <strong><span id="object">Object</span></strong
        ><br />
        <span id="country">Country</span>
        <span id="category">Category</span><br />
        <span id="year">Year</span>
      </div>
      <div
        class="rasterMoon"
        style="opacity: {$rasterOpacity.near}">
        <canvas
          width={width}
          height={height}
          id="raster-0"
        ></canvas>
        <svg 
          transform="translate({margin.l}, {margin.t})"
          width="{innerWidth}"
          height="{innerHeight}">
          <defs>
            <mask id="moon-near">
                <rect
                  x="0"
                  y="0"
                  width="{innerWidth}"
                  height="{innerHeight}"
                  fill="white"
                ></rect>
                <circle
                  cx="{innerWidth / 2}"
                  cy="{innerHeight / 2}"
                  r="{innerWidth > innerHeight ? innerHeight / 2 : innerWidth / 2}"/>
            </mask>
          </defs>
          <rect
            x="0"
            y="0"
            width="{innerWidth}"
            height="{innerHeight}"
            fill="rgb(0, 0, 0)"
            mask="url(#moon-near)"
          ></rect>
        </svg>
      </div>
  
      <div
        class="rasterMoon"
        style="opacity: {$rasterOpacity.far}">
        <canvas
          width={width}
          height={height}
          id="raster-1"
        ></canvas>
        <svg 
          transform="translate({margin.l}, {margin.t})"
          width="{innerWidth}"
          height="{innerHeight}">
          <defs>
            <mask id="moon-far">
                <rect
                  x="0"
                  y="0"
                  width="{innerWidth}"
                  height="{innerHeight}"
                  fill="white"
                ></rect>
                <circle
                  cx="{innerWidth / 2}"
                  cy="{innerHeight / 2}"
                  r="{innerWidth > innerHeight ? innerHeight / 2 : innerWidth / 2}"/>
            </mask>
          </defs>
          <rect
            x="0"
            y="0"
            width="{innerWidth}"
            height="{innerHeight}"
            fill="rgb(0, 0, 0)"
            mask="url(#moon-far)"
          ></rect>
        </svg>
      </div>
  
      <svg
        width="{window.innerWidth / 2}"
        height="{window.innerHeight}"
        class="scrolly-circles">
        <text
          x="{margin.l}"
          y="{margin.t}"
          fill="white"
          class="chart-subtitle"
          style="opacity: {$rasterOpacity.near}"
        >Near side</text>
        <text
          x="{margin.l}"
          y="{margin.t + height}"
          fill="white"
          class="chart-subtitle"
          style="opacity: {$rasterOpacity.far}"
        >Far side</text>
        <text
          x="{margin.l}"
          y="{margin.t}"
          fill="white"
          class="chart-subtitle"
          style="opacity: {waffleOpacity}"
        >{waffleText}</text>
        {#if index >= 7}
          <line
            x1="{xScale(0) - 10}"
            y1="{margin.t + 50}"
            x2="{xScale(0) - 10}"
            y2="{height * 2 - 100}"
            stroke-width="1"
            stroke="white"
            style="opacity: 0.3"
          ></line>
        {/if}
        {#if index == 7}
          {#each years.filter(d => d % 5 == 0) as year}
            <text
              class="axis-label"
              x="{xScale(0) - 20}"
              y="{yScale(year) + yScale.bandwidth() / 2}"
              fill="white"
              text-anchor="end"
            >{year}
            </text>
          {/each}
        {:else if index == 8}
          {#each countries as country}
            <text
              class="axis-label"
              x="{xScale(0) - 20}"
              y="{yScale(country) + 9}"
              fill="white"
              text-anchor="end"
            >{country}
            </text>
          {/each}
        {/if}
        <g>
          {#each $circles as {cx, cy, cr, opacity, r, g, b, id}} 
            <circle
              style="{move(cx, cy)}"
              r="{cr}"
              stroke="black"
              stroke-width="1"
              stroke-opacity="{opacity}"
              fill="rgb({r}, {g}, {b})"
              fill-opacity="{opacity}"
              id="{id}"
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
          The image below shows the far side. From Earth, we’d need to go around the Moon to reach this side.
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
</div>

<style>

  .rasterMoon {
    position: relative;
  }

  svg {
    position: absolute;
    top: 0;
    left: 0;
  }

  .tooltip__2d {
    display: none;
    position: fixed;
    top: 0;
    left: 0;
    background: rgba(255, 255, 255, 0.9);
    color: black;
    padding: 0.8em 1em;
    border-radius: 0.3em;
    z-index: 1000;
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

  .chart-subtitle {
    font-size: 1.5rem;
    font-weight: bold;
  }

  .axis-label {
    font-size: 0.6rem;
  }
</style>