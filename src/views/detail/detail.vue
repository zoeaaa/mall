<template>
  <div id="detail">
    <detail-nav-bar @titleClick="selectIndex" class="detail-nav" ref="nav"/>
    <scroll class="content1"
            ref="scroll"
            :probe-type="3"
            @scroll="contentScroll">
      <!-- 只能看到一张图片且不轮播，解决方法：v-if="topImages!=''" -->
      <!-- 属性：topImages  传入值：top-images -->

      <detail-swiper v-if="topImages!=''" :top-images="topImages"/>
      <detail-base-info :goods="goods"/>
      <detail-shop-info :shop="shop"/>
      <detail-goods-info :detail-info="detailInfo" @imageLoad="imageLoad"/>
      <detail-param-info ref="params" :param-info="paramInfo"/>
      <detail-comment-info ref="comment" :comment-info="commentInfo"/>
      <goods-list ref="recommend" :goods="recommends"/>
    </scroll>
    <back-top v-show="showBackTop" @click.native="backTop"/>
    <detail-bottom-bar @addToCart="addToCart"/>
    <toast :message="message" :show="show"/>
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
  import DetailBottomBar from './childComps/DetailBottomBar.vue'

  import Scroll from '../../components/common/scroll/Scroll.vue'

  import GoodsList from '../../components/content/goods/GoodsList.vue'
  import Toast from '../../components/common/toast/Toast.vue'


  import {backTopMixin} from "common/mixin"

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
      GoodsList,
      DetailBottomBar,
      Toast
    },
    mixins: [backTopMixin],
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
        getThemeTopY: null,
        currentIndex: 0,
        message: '',
        show: false
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
        this.themeTopYs.push(Number.MAX_VALUE)

        console.log(this.themeTopYs);
      },

      selectIndex(index) {
        //console.log(index);
        this.$refs.scroll.scrollTo(0, -this.themeTopYs[index], 500)
      },

      contentScroll(position){
        // 内容滚动，显示正确的标题
        // 1.获取y值
        const positionY = -position.y;

        let length = this.themeTopYs.length
        for (let i = 0; i < length; i++) {
          // 2.positionY和主题中值进行对比
          // [0, 13429, 14111, 14402]
          // positionY 在 0 和 13429 之间， index = 0
          // positionY 在 13429 和 14111 之间， index = 1
          // positionY 在 14111 和  14402 之间， index = 2

          // positionY 超过 14402 ， index = 3

          // if(this.currentIndex !== i && ((i < length - 1 && positionY >= this.themeTopYs[i] && positionY <
          // this.themeTopYs[i+1]) || (i === length - 1 && positionY >= this.themeTopYs[i]))){
          //   this.currentIndex = i;
          //   // console.log(this.currentIndex);
          //   this.$refs.nav.currentIndex = this.currentIndex
          // }

          // [0, 13429, 14111, 14402]
          // positionY 在 0 和 13429 之间， index = 0
          // positionY 在 13429 和 14111 之间， index = 1
          // positionY 在 14111 和  14402 之间， index = 2
          // positionY 在 14402 和一个非常大的数之间， index = 2

          if(this.currentIndex !== i && (positionY >= this.themeTopYs[i] && positionY < this.themeTopYs[i+1])){
            this.currentIndex = i;
            this.$refs.nav.currentIndex = this.currentIndex
          }
        }

        // 决定backTop按钮是否实现
        this.showBackTop = position.y <= -1000
      },

      addToCart() {
        // 1.获取购物车需要显示的信息
        const product = {}
        product.image = this.topImages[0];
        product.title = this.goods.title;
        product.desc = this.goods.desc;
        product.price = this.goods.realPrice;
        product.iid = this.iid;

        // 2.将商品添加到购物车里
        // this.$store.commit('addCart',product)
        this.$store.dispatch('addCart',product).then(res => {
          this.show = true;
          this.message = res;
          setTimeout(() => {
            this.show = false;
            this.message = ''
          },1500)
          // console.log(res);
        })
      }

    },
  }
</script>

<style>
  #detail {
    position: relative;
    z-index: 1;
    background-color: #fff;
    height: 100vh;
  }

  .detail-nav {
    position: relative;
    z-index: 9;
    background-color: #fff;
  }

  .content1 {
    background-color: #fff;
    height: calc(100% - 44px - 49px);
  }
</style>
