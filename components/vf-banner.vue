<template>
  <!-- .swiper-container>.swiper-wrapper>.swiper-slide*5 -->
  <div>
    <div :id="bphID" style="display: none;"/>
    <div :id="containerID">
      <div v-if="data && data.length > 0" :id="id" class="swiper-container">
        <div class="swiper-wrapper">
          <div v-for="(item, idx) in data" :key="idx" class="swiper-slide">
            <div class="silde-item">
              <img :src="item.path_format" alt>
              <!-- <img src="~/static/images/banner_placeholder.png" alt> -->
              <!-- <img src="~/static/images/banner-placeholder.png" alt> -->
            </div>
          </div>
        </div>
        <div v-if="data.length > 1" :class="paginationID" class="swiper-pagination"/>
        <div v-if="data.length > 1" :class="prevID" class="swiper-button-prev"/>
        <div v-if="data.length > 1" :class="nextID" class="swiper-button-next"/>
      </div>
    </div>
  </div>
</template>
<script>
import Swiper from 'swiper'

export default {
  props: {
    id: {
      type: String,
      required: false,
      default: () => {
        return ''
      }
    },
    data: {
      type: Array,
      required: {},
      default: () => {
        return []
      }
    }
  },
  computed: {
    bphID() {
      return 'banner-placeholder-' + this.id
    },
    containerID() {
      return 'banner-container-' + this.id
    },
    paginationID() {
      return 'swiper-pagination-' + this.id
    },
    prevID() {
      return 'swiper-button-prev-' + this.id
    },
    nextID() {
      return 'swiper-button-next-' + this.id
    }
  },
  mounted() {
    this.$nextTick(function() {
      this.__main()
    })
  },
  methods: {
    __main() {
      if (!this.data || this.data.length <= 0) {
        return
      }
      this.initialSwiper()
    },
    initialSwiper() {
      const me = this
      let opts = {
        direction: 'horizontal',
        clickable: true,
        observer: true, // 修改swiper自己或子元素时，自动初始化swiper
        observeParents: true, // 修改swiper的父元素时，自动初始化swiper
        nested: true,
        on: {
          init: function() {
            // swiperAnimateCache(this); // 隐藏动画元素
            // swiperAnimate(this); // 初始化完成开始动画
          },
          slideChangeTransitionEnd: function() {
            // swiperAnimate(this); // 每个slide切换结束时也运行当前slide动画
            // this.slides.eq(this.activeIndex).find('.ani').removeClass('ani'); 动画只展现一次，去除ani类名
          },
          tap: function(e) {
            const cl = e.target.classList
            if (
              cl.contains('swiper-button-prev') ||
              cl.contains('swiper-button-next')
            ) {
              return
            }
            document.querySelector('#' + me.id).classList.toggle('full-screen')
            const container = document.querySelector('#' + me.containerID)
            const bph = document.querySelector('#' + me.bphID)
            bph.style.height = '' + container.clientHeight + 'px'
            container.classList.toggle('v-mask')
            bph.classList.toggle('banner-placeholder')
          }
        }
      }
      if (this.data && this.data.length > 1) {
        opts = {
          ...opts,
          ...{
            // autoplay: true,
            pagination: {
              el: '.' + me.paginationID,
              type: 'bullets'
            },
            navigation: {
              prevEl: '.' + me.prevID,
              nextEl: '.' + me.nextID
            }
          }
        }
      }
      const swiper = new Swiper('#' + this.id, opts)

      if (swiper.realIndex === 666) {
        swiper.slideNext()
      }
    }
  }
}
</script>

<style>
@import '~/assets/css/lib/swiper.min.css';

.swiper-button-prev,
.swiper-button-next {
  background-image: url(~static/images/toolbox/back.png);
}
.swiper-button-next {
  transform: rotate(180deg);
  -webkit-transform: rotate(180deg);
  -moz-transform: rotate(180deg);
}
</style>

<style lang="scss">
/* @import '~/assets/css/lib/swiper.css'; */

.banner-placeholder {
  display: block !important;
}
.full-screen {
  top: 50%;
  transform: translateY(-50%);
  z-index: 999;
}
.swiper-container {
  width: 100%;
  @include fit2(height, 366px);
}
.swiper-slide,
.silde-item {
  position: relative;
  width: 100%;
  height: 100%;
}
.silde-item img {
  display: block;
  width: 100%;
  height: 100%;
  background-color: $placeholderColor;
}
.silde-item h1 {
  position: absolute;
  top: 0;
  @include fit(14px);
}
.swiper-pagination-bullet-active {
  background-color: $whiteColor;
}
.swiper-pagination-bullet {
  background-color: #e8e9ed;
}
</style>
