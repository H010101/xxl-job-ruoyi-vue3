<template>
  <router-view />
</template>

<script>
import useSettingsStore from '@/store/modules/settings'
import { handleThemeStyle } from '@/utils/theme'

export default {
  name: 'App',
  mounted() {
    this.$nextTick(() => {
      // 初始化主题样式
      handleThemeStyle(useSettingsStore().theme)
    })
  }
}
</script>
