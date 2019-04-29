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
        margin-top: 50px;
        text-align: left;
        margin-left: 30px;
    }
    .intro-desc {
        font-family: 'Open Sans', sans-serif;
        color: #fff;
        font-size: 20px;
        text-align:left;
        margin-left: 30px;
        margin-top: 90px;
    }
    .opioid-pills {
      width: 100%;
      margin-top: 20px;
    }
  }
 </style>
