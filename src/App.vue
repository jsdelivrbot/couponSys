<template>
  <div id="app">
    <transition name="fade" mode="out-in">
			  <router-view></router-view>
		</transition>
  </div>
</template>

<script>
export default {
  name: 'app'
}
</script>

<style lang="scss">
  @import './assets/css/base.scss';
  #app {
    .fade-enter-active,
    .fade-leave-active {
      transition: all .2s ease;
    }
    .fade-enter,
    .fade-leave-active {
      opacity: 0;
    }
  }
</style>
