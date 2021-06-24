<template>
  <div class="nav-bar">
    <!-- slot会被具体的元素替换，在slot上的属性也会被替换，所以要在slot外面包个div设置属性 -->
    <div class="left"><slot name="left"></slot></div>
    <div class="center"><slot name="center"></slot></div>
    <div class="right"><slot name="right"></slot></div>
  </div>
</template>

<script>
export default {

}
</script>

<style>
  .nav-bar {
    display: flex;
    height: 44px;
    line-height: 44px;
    text-align: center;
    box-shadow: 0 1px 1px rgba(100, 100, 100, 1);
  }

  .left, .right {
    width: 60px;
  }

  .center {
    /* flex: 1 占据除了左右两边的剩余空间*/
    flex: 1;
  }
</style>
