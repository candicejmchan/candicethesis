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

    <!-- Modal Component -->
    <b-modal ref="image-modal" size="xl" hide-footer centered title="">
      <p class="my-4">Vertically centered modal!</p>
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

export default {
  name: 'home',
  data: () => {
    return {
      introTitle: CONFIG.title,
      introDesc: CONFIG.desc,
      introBackground: CONFIG.background,
      content: CONFIG.content,
      modalContent: []
    }
  },
  created() {
    this.$on('modal-open', (d) => {
      const images1 = this.content.slice(d.index).map(k => k.content);
      const images2 = this.content.slice(0, d.index).map(k => k.content);
      this.modalContent = _.flatten(images1.concat(images2));
      this.$refs['image-modal'].show();
    })
  },
  components: {
    HelloWorld,
    Intro,
    Advert
  }
}
</script>
<style scoped lang="scss">
  .home {
    padding-left: 0px;
    padding-right: 0px;
    background-color: #000;
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
