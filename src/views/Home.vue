<template>
  <div class="home container-fluid">
    <!-- <img alt="Vue logo" src="../assets/logo.png"> -->
    <Intro :title="introTitle" :desc="introDesc" :background="introBackground" />
    <section class="row advert-section">
      <div class="col-6">
        <div class="row row-even" v-for="(advert, idx) of content" :key="idx" v-if="idx % 2 !== 0">
          <div class="col-11">
            <Advert :title="advert.title" :index="idx+1" :desc="advert.desc" :content="advert.content"/>
          </div>
        </div>
      </div>
      <div class="col-6">
        <div class="row row-odd" v-for="(advert, idx) of content" :key="idx" v-if="idx % 2 === 0">
          <div class="col-11">
            <Advert :title="advert.title" :index="idx+1" :desc="advert.desc" :content="advert.content"/>
          </div>
        </div>
      </div>
    </section>
    <section class="row specialty-chart-section">
      <div class="col-12">
        <SpecialtyChart />
      </div>
    </section>

    <!-- Modal Component -->
    <b-modal ref="image-modal" size="xl" hide-footer hide-header centered title="">
      <b-container fluid>
        <template v-for="(img, index) of modalContent" v-if="index === showImageIndex">
          <b-row>
            <b-col cols="11">
                <h3 class="float-left">{{img.index}}.</h3>
            </b-col>
            <b-col cols="1">
                <p class="modal-close-btn" v-on:click="closeModal()">x</p>
            </b-col>
          </b-row>
          <b-row>
            <b-col cols="2">
              <button class="modal-prev-btn" v-on:click="changeModalImage('prev')"><</button>
            </b-col>
            <b-col cols="5" :key="index">
              <img class="img-fluid modal-img" :src="require('../assets/images/' + img.imagelink)"/>
            </b-col>
            <b-col cols="3" :key="index">
              <p class="modal-img-desc">{{img.imagedesc}}</p>
            </b-col>
            <b-col cols="2">
              <button class="modal-next-btn" v-on:click="changeModalImage('next')">></button>
            </b-col>
          </b-row>
        </template>
      </b-container>
    </b-modal>

    <!-- carousel

    float left and right
    remove position absolute from carousel-caption
    adjust the widths -->

  </div>
</template>

<script>
import CONFIG from '@/assets/content.json'
import * as _ from 'lodash'

// @ is an alias to /src
import HelloWorld from '@/components/HelloWorld.vue'
import Intro from '@/components/intro.vue'
import Advert from '@/components/advert.vue'
import SpecialtyChart from '@/components/specialty_chart.vue'

export default {
  name: 'home',
  data: () => {
    return {
      introTitle: CONFIG.title,
      introDesc: CONFIG.desc,
      introBackground: CONFIG.background,
      content: CONFIG.content,
      modalContent: [],
      showImageIndex: 0
    }
  },
  created() {
    this.$on('modal-open', (d) => {
      const images1 = this.content.slice(d.index)
        .map( (k, index) => k.content
                .map(v => {
                  v.index = d.index + index + 1;
                  return v;
                }));
      const images2 = this.content.slice(0, d.index)
        .map( (k, index) => k.content
                  .map(v => {
                    v.index = index + 1;
                    return v}));
      this.modalContent = _.flatten(images1.concat(images2));
      this.showImageIndex = 0;
      this.$refs['image-modal'].show();
    })
  },
  methods: {
    changeModalImage(type) {
      if(type === 'next') {
        this.showImageIndex += 1;
        this.showImageIndex %= this.modalContent.length;
      } else {
        this.showImageIndex -= 1;
        this.showImageIndex = this.showImageIndex < 0 ?
            (this.modalContent.length - 1) : this.showImageIndex;
      }
    },
    closeModal() {
      this.$refs['image-modal'].hide();
    }
  },
  components: {
    HelloWorld,
    Intro,
    Advert,
    SpecialtyChart
  }
}
</script>
<style lang="scss">
  .home {
    padding-left: 0px;
    padding-right: 0px;
    background-color: #000;
  }
  .modal-content {
    .modal-body {
      background-color: #000;
      color: #fff;
      .modal-prev-btn {
        display: inline-block;
      }
      .modal-next-btn {
        display: inline-block;
      }
      .modal-img-desc {
        text-align: left;
      }
      .modal-close-btn {
        font-size: 20px;
        cursor: pointer;
        &:hover {
          color: yellow;
        }
      }
    }
  }
  .advert-section {
    margin-top: 30px;
    .row-even {
      .advert {
        margin-top:200px;
      }
    }
    .advert-3 {
      margin-top: 200px;
    }
  }

</style>
