<div class="container">
  <h1>
      <div class="change-letter"></div> on the Moon
  </h1>
  <p>
      By Yuriko Schumacher
  </p>
  <a href="#intro" class="mouse_scroll">
    <div class="mouse">
      <div class="wheel"></div>
    </div>
  </a>
</div>

<style>
  h1 {
    font-size: 3em;
    line-height: 1.2;
  }

  div.container {
      margin: 0 auto;
      height: 100vh;
      width: 80%;
      max-width: 600px;
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      text-align: center;
  }
  
  div.change-letter {
    display: inline;
    padding: 0.1em 0.3em;
    background: rgb(255, 255, 255);
    color: black;
  }

  div.change-letter::after {
    content: "Junks";
    animation: changeLetter 12s infinite;
  }
  
  @keyframes changeLetter {
    0% {
      content: "Junk";
    }
    25% {
      content: "Debris"
    }
    50% {
      content: "Man-made objects";
    }
    75% {
      content: "Mysterious hut??"
    }
  }

.mouse_scroll {
    display: block;
    margin: 0 auto;
    width: 24px;
    height: 100px;
    margin-top: 25px;
    opacity: 0;
    animation: mouse-appear 1s normal forwards ease-in-out;
    animation-delay: 5s;
    cursor: pointer;
}
.mouse {
  height: 42px;
  width: 24px;
  border-radius: 14px;
  transform: none;
  border: 2px solid rgba(255, 255, 255, 0.4);
  top: 170px;
}

.wheel {
  height: 5px;
  width: 2px;
  display: block;
  margin: 5px auto;
  background: white;
  position: relative;
  
  height: 4px;
  width: 4px;
  border: 2px solid #fff;
  -webkit-border-radius: 8px;
          border-radius: 8px;
}

.wheel {
  -webkit-animation: mouse-wheel 1s linear infinite;
  -moz-animation: mouse-wheel 1s linear infinite;
  animation: mouse-wheel 1s linear infinite;
}

@keyframes mouse-appear {
  0% {
    opacity: 0
  }
  100% {
    opacity: 1
  }
}

  @-webkit-keyframes mouse-wheel{
    0% {
      opacity: 1;
      -webkit-transform: translateY(0);
      -ms-transform: translateY(0);
      transform: translateY(0);
    }

    100% {
      opacity: 0;
      -webkit-transform: translateY(6px);
      -ms-transform: translateY(6px);
      transform: translateY(6px);
    }
  }
  @-moz-keyframes mouse-wheel {
    0% { top: 1px; }
    25% { top: 2px; }
    50% { top: 3px;}
    75% { top: 2px;}
    100% { top: 1px;}
  }
  @-o-keyframes mouse-wheel {

    0% { top: 1px; }
    25% { top: 2px; }
    50% { top: 3px;}
    75% { top: 2px;}
    100% { top: 1px;}
  }
  @keyframes mouse-wheel {

    0% { top: 1px; }
    25% { top: 2px; }
    50% { top: 3px;}
    75% { top: 2px;}
    100% { top: 1px;}
  }
</style>
