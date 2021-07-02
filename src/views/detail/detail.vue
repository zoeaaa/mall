<template>
  <div id="detail">
    <detail-nav-bar @titleClick="selectIndex" class="detail-nav"/>
    <scroll class="content1" ref="scroll">
      <!-- 只能看到一张图片且不轮播，解决方法：v-if="topImages!=''" -->
      <detail-swiper v-if="topImages!=''" :top-images="topImages"/>
      <detail-base-info :goods="goods"/>
      <detail-shop-info :shop="shop"/>
      <detail-goods-info :detail-info="detailInfo" @imageLoad="imageLoad"/>
      <detail-param-info ref="params" :param-info="paramInfo"/>
      <detail-comment-info ref="comment" :comment-info="commentInfo"/>
      <goods-list ref="recommend" :goods="recommends"/>
    </scroll>
  </div>
</template>

<script>
  import DetailNavBar from './childComps/DetailNavBar.vue'
  import DetailSwiper from './childComps/DetailSwiper.vue'
  import DetailBaseInfo from './childComps/DetailBaseInfo.vue'
  import DetailShopInfo from './childComps/DetailShopInfo.vue'
  import DetailGoodsInfo from './childComps/DetailGoodsInfo.vue'
  import DetailParamInfo from './childComps/DetailParamInfo.vue'
  import DetailCommentInfo from './childComps/DetailCommentInfo.vue'

  import Scroll from '../../components/common/scroll/Scroll.vue'
  import GoodsList from '../../components/content/goods/GoodsList.vue'

  import {getDetail,Goods,Shop,GoodsParam,getRecommend} from "network/detail"

  export default {
    name: "Detail",
    components: {
      DetailNavBar,
      DetailSwiper,
      DetailBaseInfo,
      DetailShopInfo,
      DetailGoodsInfo,
      DetailParamInfo,
      DetailCommentInfo,
      Scroll,
      GoodsList
    },
    data() {
      return {
        iid: null,
        topImages: [],
        goods: {},
        shop: {},
        detailInfo: {},
        paramInfo: {},
        commentInfo: {},
        recommends: [],
        themeTopYs: [],
        getThemeTopY: null
      }
    },
    created() {
      // 1.保存传入的iid
      this.iid = this.$route.params.iid

      // 2.根据iid请求详情数据
      getDetail(this.iid).then(res => {

        // console.log(res);
        const data = res.result
        // 1.获取顶部的图片轮播数据
        this.topImages = data.itemInfo.topImages

        // 2.获取商品信息
        this.goods = new Goods(data.itemInfo,data.columns,data.shopInfo.services)

        // 3.创建店铺信息的对象
        this.shop = new Shop(data.shopInfo)

        // 4.保存商品的详情数据
        this.detailInfo = data.detailInfo;

        // 5.获取参数的信息
        this.paramInfo = new GoodsParam(data.itemParams.info,data.itemParams.rule)

        //6. 取出评论的信息
        if(data.rate.cRate !== 0) {
          this.commentInfo = data.rate.list[0]
        }

        // this.$nextTick(() => {
          //根据最新的数据，对应的DOM是已经被渲染出来
          // 但是图片依然是没有加载完（目前获取到offserTop不包含其中的图片）
          // offserTop值不对的时候，都是因为图片的问题
        //   this.themeTopYs = []
        //   this.themeTopYs.push(0);
        //   this.themeTopYs.push(this.$refs.params.$el.offsetTop+304)
        //   this.themeTopYs.push(this.$refs.comment.$el.offsetTop+304)
        //   this.themeTopYs.push(this.$refs.recommend.$el.offsetTop+304)

        //   console.log(this.themeTopYs);

        // })


     })

      // 3.请求推荐数据
      getRecommend().then(res => {
        this.recommends = res.data.list
      })

      // 4.给getThemeTopY赋值（对给this.themeTopYs赋值的操作进行防抖）
        // 在本代码中好像不用进行防抖
      // this.getThemeTopY = debounce(() => {
          // this.themeTopYs = []
          // this.themeTopYs.push(0);
          // this.themeTopYs.push(this.$refs.params.$el.offsetTop)
          // this.themeTopYs.push(this.$refs.comment.$el.offsetTop)
          // this.themeTopYs.push(this.$refs.recommend.$el.offsetTop)

          // console.log(this.themeTopYs);
      // })
    },
    mounted() {

    },
    // 调用次数太多
    // updated() {
      // this.themeTopYs = []
      // this.themeTopYs.push(0);
      // this.themeTopYs.push(this.$refs.params.$el.offsetTop)
      // this.themeTopYs.push(this.$refs.comment.$el.offsetTop)
      // this.themeTopYs.push(this.$refs.recommend.$el.offsetTop)

      // console.log(this.themeTopYs);
    // },
    methods: {
      imageLoad() {
        this.$refs.scroll.refresh()

        this.themeTopYs = []
        this.themeTopYs.push(0);
        this.themeTopYs.push(this.$refs.params.$el.offsetTop)
        this.themeTopYs.push(this.$refs.comment.$el.offsetTop)
        this.themeTopYs.push(this.$refs.recommend.$el.offsetTop)

        console.log(this.themeTopYs);
      },
      selectIndex(index) {
        //console.log(index);
        this.$refs.scroll.scrollTo(0, -this.themeTopYs[index], 500)
      }
    },
  }
</script>

<style>
  #detail {
    position: relative;
    z-index: 9;
    background-color: #fff;
    height: 100vh;
  }

  .detail-nav {
    position: relative;
    z-index: 9;
    background-color: #fff;
  }

  .content1 {
    height: calc(100% - 44px);
  }
</style>
