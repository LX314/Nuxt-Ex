<template>
  <div>
    <template v-if="!$store.getters['center/getReviewsList'] || $store.getters['center/getReviewsList'].length <= 0">
      <NoData />
    </template>
    <template v-else>
      <div v-for="item in $store.getters['center/getReviewsList']" :key="item.key">
        <Review v-if="item.category === 'review'" :data="item"/>
        <News v-if="item.category === 'news'" :data="item"/>
      </div>
    </template>
  </div>
</template>

<script>
import Review from '~/components/vf-review'
import News from '~/components/vf-news'
import NoData from '~/components/vf-no-data'
import { mapState, mapMutations } from 'vuex'

export default {
  components: {
    Review,
    News,
    NoData
  },
  data() {
    return {
      mescroll: null
    }
  },
  computed: {
    ...mapState({
      dreviews: state => state.center.dreviews
    }),
    ...mapMutations({
      setReviewsCache: 'center/setReviewsCache'
    })
  },
  watch: {
    'mescroll.optUp.page.num'(v) {
      const me = this
      if (me._inactive) {
        return
      }
      if (parseInt(v) > 0) {
        // this.$route.query.page = v
        me.$router.push({
          path: me.$route.path,
          query: {
            ...me.$route.query,
            page: v
          }
        })
        me.$store.commit('center/setReviewsParams', {
          page: v
        })
      }
    }
  },
  async fetch({ query, store }) {
    if (store.state.center.dreviews.cache) {
      return
    }
    const p = {
      category: 'mine',
      member_id: query.member_id,
      page: query.page || store.state.center.dreviews.params.page || 1
    }
    await store.dispatch('center/reviewsList', p)
  },
  activated() {
    console.log('reviews activated')
    const me = this
    if (window.mescroll) {
      me.mescroll = window.mescroll
      me.mescroll.optUp.callback =
        me.loadReviewsData || me.methods.loadReviewsData

      const page = parseInt(me.$store.state.center.dreviews.params.page)
      if (page > 0) {
        me.mescroll.setPageNum(page + 1)
        me.$router.push({
          path: me.$route.path,
          query: {
            ...me.$route.query,
            page: page
          }
        })
      }
    }
  },
  deactivated() {
    this.mescroll && (this.mescroll.optUp.callback = null)
  },
  mounted() {
    this.$nextTick(function() {
      this.$store.commit('center/setReviewsCache', true)
      this.__main()
    })
  },
  methods: {
    __main() {},
    loadReviewsData(page) {
      const me = this
      console.log('reviews 上拉事件: ', page, '\t\tthis: ', this)
      if (page.num <= 1) {
        this.$store.commit('center/resetReviews')
      }
      const p = {
        category: 'mine',
        member_id: this.$route.query.member_id,
        page: page.num
      }
      this.$store
        .dispatch('center/reviewsList', p)
        .then(({ data }) => {
          me.mescroll.endSuccess(data.list.length, data.total_page > 1)
        })
        .catch(error => {
          console.log('Error: ', error)
          me.mescroll.endErr()
        })
    }
  }
}
</script>

<style lang="scss">
.influence {
  @include fit2(padding-top padding-bottom, 14px);
  @include fit2(padding-left padding-right, 16px);
  background-color: $whiteColor;
  font-size: 0;
  .icon {
    position: relative;
    display: inline-block;
    img {
      display: block;
      @include fit2(width, 67px);
      @include fit2(height, 20px);
    }
    span {
      @include fit(12px);
      position: absolute;
      top: 50%;
      width: 100%;
      text-align: center;
      transform: translateY(-50%);
      color: $whiteColor;
    }
  }
  .influence-content {
    @include fit2(padding-top, 12px);
    .item {
      @include fit2(margin-bottom, 8px);
      @include fit2(padding-top, 8px);
      @include fit2(padding-bottom, 10px);
      @include fit2(padding-left padding-right, 16px);
      @include fit2(border-radius, 4px);
      background-color: $placeholderColor;
      .value {
        display: block;
        color: $darkBlueColor;
        @include fit(14px);
        font-weight: bold;
      }
      .name {
        display: block;
        color: $grayColor;
        @include fit(14px);
      }
    }
  }
  .basic-information {
    @include fit2(padding-top, 24px);
    color: $blackColor;
    .title {
      @include fit(12px);
      @include fit2(padding-bottom, 4px);
      display: block;
      font-weight: bold;
    }
    .item {
      @include fit2(padding-top padding-bottom, 12px);
      display: block;
      .logo {
        @include fit2(width, 13px);
        @include fit2(height, 14px);
      }
      .value {
        @include fit(14px);
        @include fit2(padding-left, 10px);
      }
      a.value {
        text-decoration: none;
        color: $darkBlueColor;
      }
    }
  }
  .introduction {
    color: $blackColor;
    .title {
      @include fit(12px);
      @include fit2(padding-top, 14px);
      @include fit2(padding-bottom, 8px);
      display: block;
      font-weight: bold;
    }
    p {
      @include fit(14px);
    }
  }
}
</style>
