<template>
  <span class="common" :class="classMap[type]" v-show="type>=0"></span>
</template>

<script type="text/ecmascript-6">
  export default {
      props: {
          type: {
              type: Number
          },
          size: {
              type: Number
          }
      },
      created () {
          var map = ['decrease', 'discount', 'special', 'invoice', 'guarantee'];
          this.classMap = map.map(item => item + '_' + this.size);
      }
  };
</script>
<style lang="stylus" rel="stylesheet/stylus" type="text/stylus">
  @import "../../common/stylus/mixin.styl"
  .common
    display : inline-block
    vertical-align : top
    background-repeat : no-repeat
    &.decrease_1
      bg-image('decrease_1')
    &.discount_1
      bg-image('discount_1')
    &.guarantee_1
      bg-image('guarantee_1')
    &.special_1
      bg-image('special_1')
    &.invoice_1
      bg-image('invoice_1')
    &.decrease_2
      bg-image('decrease_2')
    &.discount_2
      bg-image('discount_2')
    &.guarantee_2
      bg-image('guarantee_2')
    &.special_2
      bg-image('special_2')
    &.invoice_2
      bg-image('invoice_2')
    &.decrease_3
      bg-image('decrease_3')
    &.discount_3
      bg-image('discount_3')
    &.guarantee_3
      bg-image('guarantee_3')
    &.special_3
      bg-image('special_3')
    &.invoice_3
      bg-image('invoice_3')
</style>
