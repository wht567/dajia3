<template lang="pug">
.tools
  a-menu.menu(mode="inline" theme="light" v-model:openKeys="menu_open_keys")
    a-sub-menu(key="new-shop-setting" title="新店设置")
      a-menu-item(key="new-shop-setting:mt") 
        router-link(:to="{name: 'tools-fresh-mt'}") 美团
      a-menu-item(key="new-shop-setting:elm") 
        router-link(:to="{name: 'tools-fresh-elm'}") 饿了么
    a-menu-item(key="tests:sync/del") 
      router-link(:to="{name: 'tools-tests'}") 测试产品
  router-view
</template>

<script>
export default {
  name: 'tools',
  data() {
    return {
      menu_open_keys: ['new-shop-setting']
    }
  }
}
</script>

<style lang="sass" scoped>
.tools
  display: flex
  width: 100%
  height: calc(100vh - 48px)

.menu
  width: 200px
  flex: 0 1 200px
</style>