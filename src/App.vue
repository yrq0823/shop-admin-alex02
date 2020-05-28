<template>
  <div id="app">
    <router-view></router-view>
  </div>
</template>

<script>
export default {
  data () {
    return {}
  }
}
</script>

<style lang='scss'>
  * {
    margin: 0;
    padding: 0;
  }
  html,
  body,
   #app {
    width: 100%;
    height: 100%;
      .el-breadcrumb {
      border-bottom: 1px solid #ccc;
      height: 40px;
      line-height: 40px;
      font-size: 14px;
      margin-bottom: 10px;
    }
  }
</style>
