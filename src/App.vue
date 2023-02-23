<template>
  <div id="app">
   <load3-dmodel />
    <div class="user-wrap">
      <img class="user-logo" src="@/assets/user-logo.jpg">
      <div class="user-name">by 南鸢.°</div>
    </div>
  </div>
</template>

<script>

import load3Dmodel from "@/views/load3Dmodel";
export default {
  name: 'App',
  components: {
    load3Dmodel
  }
}
</script>

<style>
html,body{
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100%;
}
#app {
  width: 100%;
  height: 100%;
}

.user-wrap {
  position: fixed;
  bottom: 20px;
  right: 20px;
  display: flex;
  align-items: center;
  color: white;
}

.user-logo {
  width: 30px;
  height: 30px;
  border-radius: 50%;
  margin-right: 10px;
}
</style>
