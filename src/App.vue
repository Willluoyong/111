<template>
  <div id="app">
    <div class="left">
      <router-link to='/'>商品列表</router-link>
      <router-link to='/second'>excle上传列表</router-link>
    </div>
    <router-view/>
  </div>
</template>

<script>
export default {
  name: 'App'
}
</script>

<style>
  #app{
    display: flex;
    flex-direction: row;

  }
  .left{
    width: 200px;
    height: 1000px;
    background-color: blue;
    display: flex;
    flex-direction: column;

  }
  .left li{
    list-style: none;
    height: 30px;
    line-height: 30px;
    border: solid 1px blueviolet;
    padding-left: 10px;
  }
</style>
