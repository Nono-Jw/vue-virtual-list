<script setup>
// This starter template is using Vue 3 <script setup> SFCs
// Check out https://v3.vuejs.org/api/sfc-script-setup.html#sfc-script-setup
import VirtualList from './components/VirtualList.vue';

const list = new Array(100).fill(0).map((_item, index) => {
  return {
    title: `标题${index}`,
    height: Math.floor(Math.random() * (100 - 50 + 1)) + 50,
  };
});
console.log(list);
</script>

<template>
  <div class="content">
    <VirtualList :list="list" :auto-height="true" :item-height="50">
      <template #default="scope">
        <div :style="{ height: scope.item.height + 'px', lineHeight: scope.item.height + 'px' }">内容</div>
      </template>
    </VirtualList>
  </div>
</template>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
body {
  margin: 0;
  padding: 0;
}
.content {
  width: 100%;
  height: 100vh;
}
</style>
