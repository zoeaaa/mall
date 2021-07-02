<template>
  <div id="home">
    <nav-bar class="home-nav"><div slot="center">购物街</div></nav-bar>
    <tab-control   :titles="['流行','新款','精选']"
                    @tabClick="tabClick"
                    ref="tabControl1"
                    class="tab-control"
                    v-show="isTabFixed"/>

    <scroll class="content"
            ref="scroll"
            :probe-type="3"
            @scroll="contentScroll"
            :pull-up-load="true"
            @pullingUp="loadMore">
      <home-swiper :banners="banners" @swiperImageLoad="swiperImageLoad"/>
      <recommend-view :recommends="recommends"/>
      <feature-view/>
      <tab-control class="tab-control"
                    :titles="['流行','新款','精选']"
                    @tabClick="tabClick"
                    ref="tabControl2"/>
      <goods-list :goods="showGoods"/>
    </scroll>

    <back-top @click.native="backClick" v-show="isShowBackTop"></back-top>

    <ul>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
    </ul>
  </div>
</template>

<script>
  import NavBar from 'components/common/navbar/NavBar.vue'
  import HomeSwiper from './childComps/HomeSwiper.vue'
  import RecommendView from './childComps/RecommendView.vue'
  import FeatureView from './childComps/FeatureView.vue'

  import TabControl from '../../components/content/tabControl/TabControl.vue'
  import GoodsList from '../../components/content/goods/GoodsList.vue'
  import Scroll from '../../components/common/scroll/Scroll.vue'
  import BackTop from '../../components/content/backTop/BackTop.vue'

  import {getHomeMultidata, getHomeGoods} from 'network/home'

  export default {
    components: {
      NavBar,
      HomeSwiper,
      RecommendView,
      TabControl,
      FeatureView,
      GoodsList,
      Scroll,
      BackTop
    },
    data() {
      return {
        // result保存created中的res（created调用完后垃圾回收会销毁res）
        // result: null
        banners: [],
        recommends: [],
        goods: {
          'pop': {page: 0, list: []},
          'new': {page: 0, list: []},
          'sell': {page: 0, list: []},
        },
        currentType: 'pop',
        isShowBackTop: false,
        tabOffsetTop: 0,
        isTabFixed: false,
        saveY: 0
      }
    },
    computed: {
      showGoods() {
        return this.goods[this.currentType].list
      }
    },

    // 让home中的内容保持原来的位置
    destroyed() {
     console.log('home destroyed');
    },
    activated() {
      this.$refs.scroll.scrollTo(0,this.saveY,10)
      this.$refs.scroll.refresh()
    },
    deactivated() {
      this.saveY = this.$refs.scroll.getScrollY()
    },
    created() {
      // 1. 请求多个数据
      this.getHomeMultidata()

      // 2.请求商品数据
      this.getHomeGoods('pop')
      this.getHomeGoods('new')
      this.getHomeGoods('sell')


    },
    mounted() {
      //1.监听item中图片加载完成
      const refresh = this.debounce(this.$refs.scroll.refresh,200)
      this.$bus.$on('itemImageLoad', () => {
        refresh()
      })


    },


    methods: {
      /**
       * 事件监听的方法
       */
      debounce(func,delay) {
        let timer = null
        return function (...args) {
          if(timer) clearTimeout(timer)
          timer = setTimeout(() => {
            func.apply(this, args)
          },delay)
        }
      },
      tabClick(index){
        switch(index) {
          case 0:
            this.currentType = 'pop'
            break
          case 1:
            this.currentType = 'new'
            break
          case 2:
            this.currentType = 'sell'
            break
        }

        // 让两个TabContol的currentIndex保持一致
        this.$refs.tabControl1.currentIndex = index;
        this.$refs.tabControl2.currentIndex = index;
      },

      backClick(){
        this.$refs.scroll.scrollTo(0,0)
      },

      contentScroll(position) {
        // 1.判断BackTop是否显示
        this.isShowBackTop = (-position.y) > 1000

        // 2.决定tabControl是否吸顶（position:fixed）
        this.isTabFixed = (-position.y) > this.tabOffsetTop
      },

      loadMore(){
        this.getHomeGoods(this.currentType)
      },

      swiperImageLoad() {
        //获取tabControl的offsetTop
        // 所有的组件都有一个属性$el,用于获取组件中的元素
        this.tabOffsetTop = this.$refs.tabControl2.$el.offsetTop;
      },

      /**
       * 网络请求相关的方法
       */
      getHomeMultidata() {
        getHomeMultidata().then(res => {
          // console.log(res);
          // this.result = res;
          this.banners = res.data.banner.list;
          this.recommends = res.data.recommend.list;
        })
      },
      getHomeGoods(type){
        // 一开始进来+1是加给刚定义的要发送给后端的page，并不是加给goods里的page
        const page = this.goods[type].page + 1
        getHomeGoods(type,1).then(res => {
          this.goods[type].list.push(...res.data.list)
          this.goods[type].page += 1

          // 完成上拉加载更多
          this.$refs.scroll.finishPullUp()
        })
      }
    },
  }
</script>

<style scoped>
  #home {
    /* padding-top: 44px; */
    height: 100vh;
    position: relative;
  }

  .home-nav{
    background-color: var(--color-tint);
    color: white;

    /* 在使用浏览器原生滚动时，为了让导航不跟随一起滚动 */
    /* position: fixed;
    left: 0;
    right: 0;
    top: 0;
    z-index: 9; */
  }

  /* .tab-control {
    //sticky: 到达固定位置就黏住不动
    position: sticky;
    top: 44px;
    z-index: 9;
  } */

  .tab-control {
    position: relative;
    z-index: 9;
  }

  .content {
    /* height: 300px; */
    overflow: hidden;

    position: absolute;
    top: 44px;
    bottom: 49px;
    left: 0;
    right: 0;
  }

</style>
