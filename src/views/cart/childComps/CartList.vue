<template>
  <div class="cart-list">
    <scroll class="content1" ref="scroll">
      <cart-list-item
          v-for="(item,index) in cartList"
          :key="index"
          :itemInfo="item"/>
    </scroll>
  </div>
</template>

<script>
  import { mapGetters } from 'vuex'
  import Scroll from '../../../components/common/scroll/Scroll.vue'
  import CartListItem from './CartListItem.vue'

  export default {
    components: {
      Scroll,
      CartListItem
    },
    computed: {
      ...mapGetters(['cartList'])
    },
    activated() {
      this.$refs.scroll.refresh()
    }
  }
</script>

<style>
  .cart-list {
    height: calc(100% - 44px - 49px - 40px);
  }

  .content1 {
    height: 100%;
    overflow: hidden;
  }
</style>
