<template>
  <div class="d-followers">
    <div class="v-follow-list">
      <div v-for="(item, idx) in list" :key="idx" class="list-item">
        <Avatar :data="item.avatar"/>
        <button :class="{'unfollow': item.f_is_follow === 0}">
          Follow
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import Avatar from '~/components/vf-avatar'
export default {
  components: {
    Avatar
  },
  props: {
    list: {
      type: Array,
      default: () => []
    }
  }
}
</script>

<style lang="scss">
.d-followers {
  .v-follow-list {
    .list-item {
      position: relative;
      .unfollow {
        color: $blackColor;
        background-color: $placeholderColor;
      }
      button {
        @include fit(12px);
        @include fit2(right, 16px);
        @include fit2(border-radius, 4px);
        @include fit2(padding-top padding-bottom, 7px);
        @include fit2(padding-left padding-right, 12px);
        background-color: $brightBlueColor;
        color: $whiteColor;
        font-weight: bold;
        position: absolute;
        top: 50%;
        transform: translateY(-50%);
        width: auto;
      }
    }
  }
}
</style>
