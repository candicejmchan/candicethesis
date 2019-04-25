<template>
  <section class="intro">
    <div class="row">
      <div class="col-7">
        <h1 class="intro-title">{{title}}</h1>
        <p class="intro-desc">{{desc}}</p>
      </div>
      <div class="col-5">
        <img alt="opioid pills" class="opioid-pills img-fluid"
             src="../assets/images/opioid_pills.png">
      </div>
    </div>
    <div class="row">
      <div class="col-12">
        <div class="quote">
          <p class="quote-text">
            “ In your mind, you can be O.K.; you can just do one. But one too many, and a thousand is never enough.“
          </p>
          <p class="quote-author">- Bruce Cherry, 58, Pennsylvania</p>
        </div>
        <img alt="yellow smoke" class="black-yellow-smoke img-fluid"
             src="../assets/images/black_and_yellow_smoke.png"/>
      </div>
    </div>
  </section>
</template>

<script>

import imgsrc from '../assets/images/black_and_yellow_smoke.png';

export default {
  name: 'Intro',
  props: {
    title: String,
    desc: String,
    background: String
  },
  data: () => {
    return {
      imgsrc: imgsrc
    }
  },
  watch: {
    background: function(prev, current) {
      console.log('################');
      console.log(current);
    }
  }
}


</script>
<style scoped lang="scss">
  .intro {
    background-color: #000;
    .intro-title {
        font-family: 'Lora', serif;
        color: #fff;
        font-size: 40px;
        margin-top: 20px;
        text-align: left;
        margin-left: 30px;
    }
    .intro-desc {
        font-family: 'Open Sans', sans-serif;
        color: #fff;
        font-size: 20px;
        text-align:left;
        margin-left: 30px;
        margin-top: 70px;
    }
    .opioid-pills {
      width: 100%;
      margin-top: 20px;
    }
    .black-yellow-smoke {
      width: 100%;
      opacity:0.5;
    }
    .quote {
      color: #fff;
      position: absolute;
      top: 450px;
      left: 18%;
      font-size: 25px;
      width: 70%;
    }
  }
 </style>
