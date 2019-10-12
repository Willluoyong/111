<template>
  <div id="app">
    <div class="left">
      <li>全部仓库合计</li>
      <li>香港仓库</li>
      <li>深圳需求待发仓库</li>
      <li>真实买家需求库</li>
      <li>陈敬军</li>
      <li>东方嘉盛仓</li>
      <li>线上刷货</li>
      <li>维科特仓库</li>
      <li>免税店刷货（欧美）</li>
      <li>欧美刷货</li>
      <li>华盛天成仓库</li>
      <li>考拉供货仓库</li>
      <li>美铺供货仓库</li>
      <li>免税店刷货（韩汉）</li>
      <li>免税店刷货未娶货库</li>
      <li>韩国仓库</li>
      <li>香港仓库</li>
      <li>残次仓库</li>
      <li>梁凯欧美妆库</li>
      <li>韩国韵达库</li>
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

  }
  .left li{
    list-style: none;
    height: 30px;
    line-height: 30px;
    border: solid 1px blueviolet;
    padding-left: 10px;
  }
</style>
