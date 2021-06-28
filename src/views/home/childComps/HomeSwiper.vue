<template>
  <swiper>
    <swiper-item :key="item.id" v-for="item in banners">
      <a :href="item.link">
        <img :src="item.image" alt="" @load="imageLoad">
      </a>
    </swiper-item>
  </swiper>

</template>

<script>
  import {Swiper, SwiperItem} from 'components/common/swiper'

  export default {
    props: {
      banners: {
        type: Array,
        default() {
          return []
        }
      }
    },
    data() {
      return {
        isLoad: false
      }
    },
    components: {
      Swiper,
      SwiperItem
    },
    methods: {
      imageLoad() {
        // 这里不进行多次调用和debounce的区别:
        //    debounce是防止频繁调用，而这里是只需要调用一次就行，多的浪费性能
        if (!this.isLoad) {
          this.$emit('swiperImageLoad');
          this.isLoad = true
        }
      }
    },
  }
</script>

<style>

</style>
