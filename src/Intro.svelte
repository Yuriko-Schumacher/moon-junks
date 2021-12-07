<script>
  import * as d3 from "d3"
  import Scroller from "@sveltejs/svelte-scroller";
  import Title from "./Components/Title.svelte"

  let index, offset, progress;
  const imgSize = { w: window.innerWidth / 4 }
  let backGround, src, alt;
  let cap = {
    url: "",
    text: ""
  }

  $: {
    console.log(index)
    if (index <= 1) {
      backGround = d3.select(".background-img")
      backGround.style("margin", "0 0 0 auto")
      src = "./image/moon-phases.gif"
      alt = "Moon phases"
      cap = {
        url: "https://moon.nasa.gov/moon-in-motion/moon-phases/",
        text: "NASA"
      }
    } else if (index === 3) {
      src = ""
      alt = ""
      cap = {
        url: "",
        text: ""
      }
    } else if (index === 4) {
      backGround = d3.select(".background-img")
      backGround.style("margin", "auto auto 0 0")
      src = "./image/bezos_edited.png"
      alt = "Jeff Bezos"
      cap = {
        url: "https://www.flickr.com/photos/georgewbushcenter/41061224225",
        text: "Creative Commons"
      }
    }
  }
</script>

<Scroller top="{0}" bottom="{1}" threshold="{0.8}" bind:index bind:offset bind:progress>
  <div class="background" slot="background">
    <div class="background-img">
      <img
      width="{imgSize.w}"
      src="{src}"
      alt="{alt}">
      <div class="caption">
        <a href="{cap.url}" target="_blank">{cap.text}</a>
      </div>
    </div>
  </div>

  <div class="foreground" slot="foreground">
    <section data-section-id="0"><Title /></section>
    <section data-section-id="1" id="intro"></section>
    <section data-section-id="2">
      <article>
        <h2>
          A mysterious object spotted
        </h2>
        <p>
          China's Moon rover Yutu 2 has spotted a mysterious cubic object on the far side of the Moon. Our Space, a website affiliated with the China National Space Administration, <a href="https://mp.weixin.qq.com/s/dpKkjWHk9tay3JERnrfuUw" target="_blank">reported Friday</a> that scientists observed the object, introducing it as "神秘小屋" ("mystery hut").
        </p><br>
        <img src="./image/mystery-house.jpeg" width="100%" alt="">
        <div class="in-line-caption">The image that captured a cubic object. (Credit: CNSA/Our Space)</div><br>
        <p>
          According to <a href="https://www.space.com/china-yutu-2-moon-rover-cube-shaped-object-photos" target="_blank">space.com</a>, Yutu 2 will spend the next two to three lunar days (two to three Earth months) to get a closer look. Space enthusiasts got excited with the news, whether there is a possibility of eilians or not. But <a href="https://interestingengineering.com/cube-shaped-object-on-the-far-side-of-the-moon-not-aliens" target="_blank">some news reports</a> pointed out the object was not placed by any kinds of aliens, adding that an impact could have created it or it’s just a scattered debri.
        </p>
        <p>
          It is indeed mysterious though, since there have been fewer human landing on the lunar far side. According to NASA’s <a href="https://history.nasa.gov/FINAL%20Catalogue%20of%20Manmade%20Material%20on%20the%20Moon.pdf" target="_blank">Catalogue of Manmade Material on the Moon</a>, which offers a comprehensive list of human artifacts and landing sites on the Moon, by 2012, there had been more than 800 man-made objects left on the Moon. The far side has been less touched by humans than the near side -- the only side that is visible by the earth.
        </p>
        <p>
          Of course, since then, countries have conducted more space missions. China’s Cheng’e 4, which the rover Yutu 2 formed a part of, is just one of them.
        </p>   
      </article>
    </section>
    <section data-section-id="3"></section>
    <section data-section-id="4">
      <article>
        <h2>
          Send polluting industries to space to protect environment?
        </h2>
        <p>
          Moreover, the race to get tourists to space is heating up. Blue Origin, one of the major aerospace companies founded by billionaire Jeff Bezos, completed its first human flight with citizens this year. As one of the crews, Bezos expressed his excitement with the success, though his post-flight comment aroused controversy.
        </p>
        <p>
          “We need to take all heavy industry, all polluting industry, and move it into space” in order to keep earth as a beautiful planet, Bezos said <a href="https://www.nbcnews.com/science/science-news/jeff-bezos-says-spaceflight-reinforced-commitment-solving-climate-chan-rcna1467" target="_blank">during an interview with NBC</a>. “This is the very beginning of that, and it’s exciting,” he added.
        </p>
        <p>
          Yes, the planet earth is in a crisis of environmental pollution and climate change. But would it justify an attempt to send pollutants into space?
        </p> 
      </article>
    </section>
  </div>
</Scroller>

<style>
  section {
    margin: 0;
  }

  .background {
    height: 100vh;
    display: flex;
  }

  .background-img {
    width: 25vw;
    max-height: 50vh;
    margin: 5rem 0;
  }

  .background-img img {
    margin-top: 5rem;
  }

  .caption {
    font-size: 0.7em;
    margin: 0 3rem 3rem 0;
    text-align: right;
    font-style: italic;
    opacity: 0.5;
  }

  .caption a {
    color: white;
  }

  .in-line-caption {
    font-size: 0.7em;
    margin: 0;
    text-align: right;
    font-style: italic;
    opacity: 0.5;
  }

  h2 {
    margin-top: 5rem;
  }
</style>