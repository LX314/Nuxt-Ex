<template>
  <div>
    <template v-if="!$store.getters['center/getPostList'] || $store.getters['center/getPostList'].length <= 0">
      <NoData />
    </template>
    <template v-else>
      <div v-for="item in $store.getters['center/getPostList']" :key="item.key">
        <Post v-if="item.f_category === 'posts'" :key="item.key" :item="item"/>
        <Challenge v-if="item.f_category === 'challenge'" :key="item.key" :item="item"/>
        <Repost v-if="item.f_category === 'repost'" :key="item.key" :item="item"/>
      </div>
    </template>
  </div>
</template>

<script>
import Post from '~/components/vf-post'
import Repost from '~/components/vf-repost'
import Challenge from '~/components/vf-challenge'
import NoData from '~/components/vf-no-data'
import { mapState } from 'vuex'

export default {
  components: {
    Post,
    Repost,
    Challenge,
    NoData
  },
  data() {
    return {
      mescroll: window.mescroll
    }
  },
  computed: {
    ...mapState({
      param: state => state.center.dpost.params
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
        me.$store.commit('center/setPostParams', {
          page: v
        })
      }
    }
  },
  async fetch({ query, param, store }) {
    if (store.state.center.dpost.cache) {
      return
    }
    const p = {
      type: query.type || 'all',
      member_id: query.member_id,
      page: query.page || store.state.center.dpost.params.page || 1
    }
    await store.dispatch('center/memberPost', p)
  },
  mounted() {
    this.$nextTick(function() {
      this.$store.commit('center/setPostCache', true)
      this.__main()
    })
  },
  activated() {
    console.log('post activated')
    const me = this
    if (window.mescroll) {
      me.mescroll = window.mescroll
      me.mescroll.optUp.callback = me.loadPostData || me.methods.loadPostData

      const page = parseInt(me.$store.state.center.dpost.params.page)
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
  methods: {
    __main() {},
    loadPostData(page) {
      const me = this
      console.log('post 上拉事件: ', page, '\t\tthis: ', this)
      const query = this.$route.query
      const p = {
        type: query.type || 'all',
        member_id: this.$route.query.member_id,
        page: page.num
      }
      this.$store
        .dispatch('center/memberPost', p)
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
