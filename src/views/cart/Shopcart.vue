<template>
  <div class="cart">
    <!-- 导航 -->
    <nav-bar class="nav-bar1">
      <div slot="center">购物车({{length}})</div>
    </nav-bar>

    <!-- 商品的列表 -->
    <cart-list/>

    <!-- 底部汇总 -->
    <cart-bottom-bar/>
  </div>
</template>

<script>
  import NavBar from '../../components/common/navbar/NavBar.vue'
  import CartList from './childComps/CartList.vue'
  import CartBottomBar from './childComps/CartBottomBar.vue'

  import {mapGetters} from 'vuex'

  export default {
    components: {
      NavBar,
      CartList,
      CartBottomBar
    },
    computed: {
      // cartLength() {
      //   return this.$store.getters.cartLength
      // }

      // 两种语法
      // ...mapGetters(['cartLength','cartList'])
      ...mapGetters({
        length: 'cartLength'
      })
    }
  }
</script>

<style>
  .cart {
    height: 100vh;
  }
  .nav-bar1 {
    background-color: var(--color-tint);
    color: #fff;
    /* font-weight: 700; */
  }

</style>
