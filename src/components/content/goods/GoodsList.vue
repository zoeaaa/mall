<template>
  <div class="goods">
    <goods-list-item
      v-for="item in goods"
      :goods-item="item" class="item">
    </goods-list-item>
  </div>
</template>

<script>
  import GoodsListItem from './GoodsListItem.vue'

  export default {
    components: {
      GoodsListItem
      },
    props: {
      goods: {
        type: Array,
        default() {
          return []
        }
      }
    }
  }
</script>

<style>
  .goods {
    display: flex;
    flex-wrap: wrap;
    /* 均等分 */
    justify-content: space-around;

    padding: 2px;
  }

  .goods .item {
    width: 48%;
  }
</style>
