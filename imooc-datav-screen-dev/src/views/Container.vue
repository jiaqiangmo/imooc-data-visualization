<template>
  <div class="container-page">
     <imooc-container :options="{ width: 3840, height: 2160 }">
    <!--<imooc-container>-->
      <div class="test">123456789</div>
    </imooc-container>
  </div>
</template>

<script>
export default {
  name: 'Container'
}
</script>

<style lang="scss" scoped>
.test {
  font-size: 48px;
  color: red;
}
</style>
