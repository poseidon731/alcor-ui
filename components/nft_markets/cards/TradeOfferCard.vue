<template lang="pug">
.normalcard.radius10.tradeoffercard
  header.d-flex.flex-column.justify-content-between.mb-2
    div.d-flex.justify-content-between
      div
        img(src='~/assets/images/small_shape.svg' style="width:21px;height:21px")
        span.card-title {{mint}}
      div
        img(src='~/assets/images/double_arrow.svg' alt='')
        img.card-title-img(src='~/assets/images/fire.svg' alt='')
        span.card-number # {{mintCount}}
    video.main-img.radius10(v-if='videoBackground', autoplay='true', loop='true')
      source(
        :src='"https://resizer.atomichub.io/videos/v1/preview?ipfs=" + videoBackground.video + "&size=370&output=mp4"',
        type='video/mp4'
      )
    .main-img.radius10(v-else-if='imageBackground', :style='imageBackground')
      img(:src='imageBackground.backgroundImage' alt='')
    .main-img.radius10(v-else, :style='defaultBackground')
    //- .main-img(v-if='videoBackground')
    //-   video(:class="['main-img', 'radius10', (mode === 'setsList' ? 'sets-list-mode' : '')]", autoplay='true', loop='true')
    //-     source(
    //-       :src='"https://resizer.atomichub.io/videos/v1/preview?ipfs=" + videoBackground.video + "&size=370&output=mp4"',
    //-       type='video/mp4'
    //-     )
    //- div(:class="['main-img', 'radius10', (mode === 'setsList' || mode === 'templates' ? 'sets-list-mode' : '')]", v-else-if='imageBackground', :style='imageBackground')
    //-    img(:src='imageBackground.backgroundImage' alt='')
    //- div(
    //-   :class='["main-img", "radius10", mode === "setsList" || mode === "templates" ? "sets-list-mode" : ""]',
    //-   v-else,
    //-   :style='defaultBackground'
    //- )
    //- .main-img.radius10(v-else='' :style='defaultBackground')
  //- .d-flex.justify-content-between.align-items-end.mt-2.mb-2
    p.token-info--title Sword
    p.token-group Almemes
  .btn-group.justify-content-between.pb-2(v-if="!this.data.nobtngroup")
    button.btn-border--green.mr10.radius6 Details
    button.btn-fill--green.radius6 Add
</template>

<script>
import { mapState } from 'vuex'
import defaultImg from '~/assets/images/default.png'

export default {
  props: ['data', 'mint', 'mintCount', 'imageBackground', 'videoBackground', 'mode'],

  data() {
    return {
      search: '',
      defaultBackground: {
        backgroundSize: 'cover',
        backgroundImage: 'url(' + defaultImg + ')',
      },
      sellOrders: [],
    }
  },
  computed: {
  },
}
</script>

<style lang="scss">
.tradeoffercard {
  padding: 11px 13px;
  .card-title {
    color: #66C167;
    padding-left: 5px;
  }
  .card-title-img {
    margin: 0 8px;
  }
  .card-number {
    background: #212121;
    border-radius: 8px;
    padding: 2px 6px;
  }
  .token-info--title {
    color: #66C167;
    font-size: 18px;
  }
  .token-group {
    font-size: 12px;
  }
  p {
    margin: 0;
  }

  .offer-information {
    margin: 6px 0;
  }

  .offer-information .success-icon {
    width: 10px;
    height: 10px;
  }

  .radius10 {
    border-radius: 10px !important;
  }

  .radius6 {
    border-radius: 6px !important;
  }

  .mr10 {
    margin-right: 10px !important;
  }

  .normalcard {
    width: 220px;
    height: 100%;
    padding: 6px 10px;
    background-color: #202021;
  }

  .btn-border--green {
    width: 108px;
    height: 33px;
    color: #fff;
    background-color: #161617;
    border: 1px solid #67C23A;
    font-size: 14px;
    padding: 5px 10px;
    font-weight: 400;
  }

  .btn-border--green:hover {
    background-color: transparent;
    color: #67C23A;
  }

  .btn-fill--green {
    color: #000;
    width: 82px;
    height: 33px;
    background-color: #67C23A;
    border: 1px solid #67C23A;
    font-size: 14px;
    font-weight: 400;
    padding: 5px 10px;
  }

  .smaller-btn {
    width: 83px;
  }

  .bigger-btn {
    width: 107px;
  }

  .main-img {
    width: 194px;
    height: 250px !important;
    object-fit: cover;
    margin: auto;
  }

  .btn-fill--green:hover {
    background-color: transparent;
    color: #67C23A;
  }

  .wax-price {
    color: #f89022;
  }

  .default-price {
    color: #67C23A;
  }
}
</style>
