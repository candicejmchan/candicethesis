<template>
  <section :class="['advert', 'advert-'+index]">
    <div class="heading-desc">
      <h3 class="advert-heading">
        <span>{{index}} .</span>
        {{title}}
      </h3>
      <p class="advert-desc">{{desc}}</p>
    </div>
    <div class="plus-button" v-on:click="showModal(index)">+</div>
    <template v-for="(img, idx) of content">
      <div :class="['advert-img-container',
                    'advert-img-container-'+index,
                    'advert-img-container-'+index+'-'+idx]"
           :key="idx">
        <img class="img-fluid advert-img" :src="require('../assets/images/' + img.imagelink)"/>
      </div>
    </template>

  </section>
</template>
<script>

export default {
  name: 'Advert',
  props: {
    index: Number,
    title: String,
    desc: String,
    content: Array
  },
  data: () => {
    return {
    }
  },
  methods: {
    showModal: function(index) {
      this.$parent.$emit('modal-open', {index: index - 1});
    }
  }
}



</script>

<style scoped lang="scss">
  .advert {
    text-align:left;
    color: #fff;
    margin-left: 30px;
    .heading-desc {
      height: 200px;
      .advert-heading {
        font-family: 'Lora', serif;
        span {
          text-decoration: underline;
        }
      }
      .advert-desc {
        font-family: 'Open Sans', sans-serif;
      }
    }
    .advert-img-container {
      margin-right: 10px;
      display: inline-block;
      .advert-img {
        height:200px;
        border-radius: 20px;
      }
    }
    .advert-img-container-1-0 {
      width: 40%;
    }
    .advert-img-container-1-1 {
      width: 30%;
    }
    .advert-img-container-1-2 {
      width: 20%;
    }
    .plus-button {
      border-radius: 100px;
      background-color: #fff;
      color: #000;
      width: 45px;
      font-size: 30px;
      height: 45px;
      padding-left: 12px;
      margin-bottom: 5px;
      cursor: pointer;
      &:hover {
        background-color: #F8E368;
      }
    }
  }
</style>
