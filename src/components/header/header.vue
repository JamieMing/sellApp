<template>
  <div class="header">
    <div class="content-wrapper">
      <div class="avatar">
        <img :src="seller.avatar" alt="" width="64" height="64">
      </div>
      <div class="content">
        <div class="title">
          <span class="brand"></span>
          <span class="name">{{seller.name}}</span>
        </div>
        <div class="description">
          {{seller.description}}/{{seller.deliveryTime}}分钟送达
        </div>
        <div v-if="seller.supports" class="support">
          <GoodsTypeIcon :size="1" :type="seller.supports[0].type" class="icon"></GoodsTypeIcon>
          <span class="text">{{seller.supports[0].description}}</span>
        </div>
      </div>
      <div class="support-count" v-if="seller.supports" @click="showDetail">
        <span class="count">{{seller.supports.length}}个</span>
        <i class="icon-keyboard_arrow_right"></i>
      </div>
    </div>
    <div class="bulletin-wrapper">
      <span class="bulletin-title"></span><span class="bulletin-text">{{seller.bulletin}}</span>
      <i class="icon-keyboard_arrow_right"></i>
    </div>
    <div class="background">
      <img :src="seller.avatar" alt="" width="100%" height="100%">
    </div>
    <transition name="fade">
      <div class="detail" v-show="detailShow">
      <div class="detail-wrapper clearFix">
        <div class="detail-main">
          <h1 class="name">{{seller.name}}</h1>
          <div class="star-wrapper">
            <star :size="48" :score="seller.score"></star>
          </div>
          <div class="title">
            <div class="line"></div>
            <div class="text">优惠信息</div>
            <div class="line"></div>
          </div>
          <ul v-if="seller.supports" class="supports">
            <li class="support-item" v-for="item in seller.supports">
              <GoodsTypeIcon :size="2" :type="item.type" class="icon"></GoodsTypeIcon>
              <span class="text">{{item.description}}</span>
            </li>
          </ul>
          <div class="title">
            <div class="line"></div>
            <div class="text">商家公告</div>
            <div class="line"></div>
          </div>
          <div class="bulletin">
            <p class="content">{{seller.bulletin}}</p>
          </div>
        </div>
      </div>
      <div class="detail-close" @click="hideDetail">
        <i class="icon-close"></i>
      </div>
    </div>
    </transition>
  </div>
</template>

<script type="text/ecmascript-6">
  import Star from '../star/star.vue';
  import GoodsTypeIcon from '../goodsTypeIcon/goodsTypeIcon.vue';
  export default {
    name: 'header',
    data () {
      return {
        seller: {},
        detailShow: false
      };
    },
    methods: {
      showDetail() {
          this.detailShow = true;
      },
      hideDetail() {
          this.detailShow = false;
      }
    },
    created() {
        //  promise链式调用
        this.$http.get('/api/seller').then((response) => {
            return response.json();
        }).then((json) => {
            this.seller = json.seller;
        });
        this.classMap = ['decrease', 'discount', 'special', 'invoice', 'guarantee'];
    },
    components: {
        star: Star,
        GoodsTypeIcon: GoodsTypeIcon
    }
  };
</script>
<style lang="stylus" rel="stylesheet/stylus" type="text/stylus">
  @import "../../common/stylus/mixin.styl"
.header
  color: #fff
  position : relative
  overflow : hidden
  background: rgba(7, 17, 27, 0.5)
  .content-wrapper
    padding : 24px 12px 18px 24px
    font-size: 0
    position : relative
    .avatar
      display : inline-block
      vertical-align : top
      img
        border-radius : 2px
    .content
      display : inline-block
      margin-left : 16px
      .title
        margin : 2px 0 8px 0
        .brand
          height: 18px
          width: 30px
          vertical-align : top
          display : inline-block
          bg-image('brand')
          background-size : 30px 18px
          background-repeat : no-repeat
        .name
          margin-left: 6px
          font-size : 16px
          line-height : 18px
          font-weight : bold

      .description
        margin-bottom : 10px
        line-height: 12px
        font-size : 12px
      .support
        .icon
          width: 12px
          height: 12px
          margin-right : 4px
          background-size: 12px 12px
        .text
          font-size: 10px
          line-height : 12px
    .support-count
      position : absolute
      right: 12px
      bottom: 18px
      padding: 0 8px
      height: 24px
      line-height: 24px
      border-radius : 14px
      background: rgba(0, 0, 0, 0.2)
      text-align : center
      .count
        font-size :10px
      .icon-keyboard_arrow_right
        font-size : 10px
        line-height : 24px
        margin-left : 2px

  .bulletin-wrapper
    position : relative
    height: 28px
    line-height : 28px
    padding: 0 22px 0 12px
    white-space : nowrap
    overflow : hidden
    text-overflow : ellipsis
    background: rgba(7, 17, 27, 0.2 )
    .bulletin-title
      display : inline-block
      vertical-align : top
      margin-top : 8px
      width: 22px
      height: 12px
      bg-image('bulletin')
      background-size: 22px 12px;
      background-repeat : no-repeat;
    .bulletin-text
      vertical-align : top
      font-size : 10px
      margin: 0 4px
    .icon-keyboard_arrow_right
      font-size : 10px
      position : absolute
      right: 12px
      top: 8px
  .background
    position : absolute
    top: 0
    left: 0
    z-index: -1
    width: 100%
    height: 100%
    filter : blur(10px)
  .detail
    position : fixed
    z-index: 100
    height: 100%
    width: 100%
    overflow : auto
    top: 0
    left: 0px
    background: rgba(7, 17, 27, 0.8)
    backdrop-filter: blur(10px)
    &.fade-enter-active, &.fade-leave-active
      transition: all 0.5s
    &.fade-enter, &.fade-leave-to
      opacity: 0
      background: rgba(7, 17, 27, 0)
    .detail-wrapper
      min-height: 100%
      width: 100%
      .detail-main
        margin-top: 64px
        padding-bottom: 64px
        .name
          line-height: 16px
          text-align: center
          font-size: 16px
          font-weight: 700
        .star-wrapper
          margin-top: 18px
          padding:2px 0
          text-align: center
        .title
          width: 80%
          margin: 28px auto 24px auto
          display: flex
          .line
            flex: 1
            position: relative
            top: -6px
            border-bottom: 1px solid rgba(255, 255, 255, 0.2)
          .text
            padding: 0 12px
            font-size: 14px
            font-weight: 700
        .supports
          width: 80%
          margin: 0 auto
          .support-item
            padding: 0 12px
            margin-bottom: 12px
            font-size: 0px
            &.last-child
              margin-bottom: 0px
            .icon
              width: 16px
              height: 16px
              margin-right: 6px
              background-size: 16px 16px
            .text
              line-height: 16px
              font-size: 12px
        .bulletin
          width: 80%
          margin: 0 auto
          .content
            padding: 0 12px
            line-height: 24px
            font-size: 12px
    .detail-close
      position: relative
      width: 32px
      height: 32px
      margin: -64px auto 0 auto
      clear: both
      font-size: 32px

</style>
