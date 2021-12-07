<script>
  import { onMount } from 'svelte';
  import * as d3 from "d3"
  import gsap from "gsap";
  import Scroller from "@sveltejs/svelte-scroller";
  import { spring } from 'svelte/motion';
  import { cubicOut } from 'svelte/easing';

  import Shaders from "./Shaders.js";

  export let data;

  const nearSideD = data.filter(d => Math.abs(d.east) < 90)

  let index, offset, progress;
  const width = 900;
  const height = 550;
  const projection = d3.geoOrthographic();

  console.log(data)

  onMount(
    async() => {
    const container = d3.select("#container")
    const canvas = container
        .select("canvas")
        .attr("width", width)
        .attr("height", height);

    const context = canvas.node().getContext("2d");

    const image = new Image;
    image.onload = onload;
    image.src = "./image/moon_lowres.jpeg";

    function onload() {
      let dx = image.width;
      let dy = image.height;

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

    const move = (cx, cy) => `transform: translate(${cx}px, ${cy}px)`;
    const colors = {
      yellow : {
        r: 255,
        g: 233,
        b: 0
      },
      white : 238,
      black : 34
    }

    let circles = spring(
      nearSideD.map((d) => ({
        cx: 0,
        cy: 0,
        cr: 0,
        strokeWidth: 0,
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

    $: if (index === 0) {
      newCircles = nearSideD.map((d) => ({
            cx: projection([d.east, d.north])[0],
            cy: projection([d.east, d.north])[1],
            cr: 5,
            strokeWidth: 0.4,
            opacity: 0.8,
            r: colors.yellow.r,
            g: colors.yellow.g,
            b: colors.yellow.b,
          }))
      circles.set(newCircles)
    }
</script>

<Scroller top="{0.2}" bottom="{0.8}" bind:index bind:offset bind:progress>
  <div class="background" slot="background">
    <div class="tooltip">
      <strong><span id="object">Object</span></strong
      ><br />
      <span id="country">Country</span>
      <span id="category">Category</span><br />
      <span id="year">Year</span>
    </div>
    <div id="container">
      <canvas></canvas>
      <svg width={width} height={height}>
        <defs>
          <mask id="moon">
              <circle id="outer" cx="480" cy="250" r="800" fill="white"/>
              <circle id="inner" cx="480" cy="250" r="250"/>
          </mask>
        </defs>
        <rect
          x="0"
          y="0"
          width="{width}"
          height="{height}"
          fill="rgb(0, 0, 0)"
          mask="url(#moon)"
        ></rect>
        <g>
          {#each $circles as {cx, cy, cr, strokeWidth, opacity, r, g, b}} 
            <circle
              style="{move(cx, cy)}"
              r="{cr}"
              stroke="#222222"
              stroke-width="{strokeWidth}"
              stroke-opacity="{opacity}"
              fill="rgb({r}, {g}, {b})"
              fill-opacity="{opacity}"
            />
          {/each}
        </g>
      </svg>
    </div>

    <p>Section {index + 1} is currently active.</p>
  </div>

  <div class="foreground" slot="foreground">
    <section>This is the first section.</section>
    <section>This is the second section.</section>
    <section>This is the third section.</section>
  </div>
</Scroller>

<style>

  #container {
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
    width: calc(100vw - 800px);
    background: rgba(64, 224, 208, 0.1);
  }
</style>