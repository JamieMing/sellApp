<template>
  <transition name="move">
    <div class="foodfx" v-show="showFlag" ref="food">
      <div class="food-content">
        <div class="image-header">
          <img :src="food.image" alt="">
          <div class="back" @click="hide">
            <i class="icon-arrow_lift"></i>
          </div>
        </div>
        <div class="content">
          <h1 class="title">{{food.name}}</h1>
          <div class="detail">
            <span class="sell-count">月售{{food.sellCount}}份</span>
            <span class="rating">好评率{{food.rating}}</span>
          </div>
          <div class="price">
            <span class="now">￥{{food.price}}</span><span v-show="food.oldPrice" class="old">￥{{food.oldPrice}}</span>
          </div>
          <div class="cartcontrol-wrapper">
            <cartcontrol :food="food" :eventHub="eventHub"></cartcontrol>
          </div>
          <transition name="fade">
            <div class="buy" @click="addFirst" v-show="!food.count || food.count === 0">加入购物车</div>
          </transition>
        </div>
        <split v-show="food.info"></split>
        <div class="info" v-show="food.info">
          <h1 class="title">商品信息</h1>
          <p class="text">{{food.info}}</p>
        </div>
        <split></split>
        <div class="rating">
          <h1 class="title">商品评价</h1>
          <ratingselect :select-type="selectType" :only-content="onlyContent" :desc="desc" :ratings="food.ratings"
          :eventHub="eventHub"></ratingselect>
          <div class="rating-wrapper">
            <ul v-show="food.ratings && food.ratings.length">
              <li v-for="rating in food.ratings" class="rating-item border-1px" v-show="needShow(rating.rateType,rating.text)">
                <div class="user">
                  <span class="name">{{rating.username}}</span>
                  <img :src="rating.avatar" alt="" width="12" height="12" class="avater">
                </div>
                <div class="time">{{rating.rateTime | formatDate}}</div>
                <p class="text">
                  <span :class="{'icon-thumb_up':rating.rateType===0,
                  'icon-thumb_down':rating.rateType===1}"></span>{{rating.text}}
                </p>
              </li>
            </ul>
            <div class="no-rating" v-show="!food.ratings || !food.ratings.length">暂无评价</div>
          </div>
        </div>
      </div>
    </div>
  </transition>
</template>

<script type="text/ecmascript-6">
  import BScroll from 'better-scroll';
  import cartcontrol from '../cartcontrol/cartcontrol.vue';
  import Vue from 'vue';
  import {formatDate} from '../../common/js/date.js';
  import split from '../split/split.vue';
  import ratingselect from '../ratingselect/ratingselect.vue';
  // const POSITIVE = 0;
  // const NEGATIVE = 1;
  const ALL = 2;
  export default {
      props: {
        food: {
          type: Object
        },
        eventHub: {
          type: Object
        }
      },
    data() {
      return {
          showFlag: false,
          selectType: ALL,
          onlyContent: false,
          desc: {
              all: '全部',
              positive: '推荐',
              negative: '吐槽'
          }
      };
    },
    created() {
          this.eventHub.$on('ratingtype.select', (type) => {
             this.selectType = type;
             this.$nextTick(() => {
               this.scroll.refresh();
             });
          });
          this.eventHub.$on('content.toggle', (onlyContent) => {
            this.onlyContent = onlyContent;
            this.$nextTick(() => {
              this.scroll.refresh();
            });
          });
    },
    filters: {
      formatDate(time) {
          let date = new Date(time);
          return formatDate(date, 'yyyy-MM-dd hh:mm');
      }
    },
     methods: {
          show() {
              this.showFlag = true;
              this.selectType = ALL;
              this.onlyContent = true;
              this.$nextTick(() => {
                 if (!this.scroll) {
                     this.scroll = new BScroll(this.$refs.food, {
                         click: true
                     });
                 } else {
                     this.scroll.refresh();
                 }
              });
          },
          hide() {
              this.showFlag = false;
          },
          addFirst(event) {
              if (!event._constructed) {
                  return;
              }
              this.eventHub.$emit('cart.add', event.target);
              Vue.set(this.food, 'count', 1);
          },
          needShow(type, text) {
              if (this.onlyContent && !text) {
                  return false;
              }
              if (this.selectType === ALL) {
                  return true;
              } else {
                  return type === this.selectType;
              }
          }
     },
    components: {
        cartcontrol,
        split,
        ratingselect
    }
  };
</script>
<style lang="stylus" rel="stylesheet/stylus" type="text/stylus">
  @import "../../common/stylus/mixin.styl"
.foodfx
  position: fixed
  left: 0
  top: 0px
  bottom: 48px
  z-index: 30
  width: 100%
  background: #fff
  transform: translate3d(0, 0, 0)
  &.move-enter-active, &.move-leave-active
    transition: all 0.2s linear
  &.move-enter, &.move-leave-to
    transform: translate3d(100%, 0, 0)
  .image-header
    position: relative
    width: 100%
    height: 0
    /*宽高相等的设置*/
    padding-top: 100%
    img
      position: absolute
      top: 0
      left: 0
      width: 100%
      height: 100%
    .back
      position: absolute
      top: 10px
      left: 0
      .icon-arrow_lift
        display: block
        padding: 10px
        font-size: 20px
        color: white
  .content
    padding: 18px
    position: relative
    .title
      line-height: 14px
      margin-bottom: 8px
      font-size: 14px
      font-weight: 700
      color: rgb(7, 17, 27)
    .detail
      margin-bottom: 18px
      line-height: 10px
      font-size: 0
      .sell-count, .rating
        font-size: 10px
        color: rgb(147, 153, 159)
      .sell-count
        margin-right: 12px

    .price
      font-weight: 700
      line-height: 24px
      .now
        margin-right: 8px
        font-size: 14px
        color: rgb(240, 20, 20)
      .old
        text-decoration: line-through
        font-size: 10px
        color: rgb(147, 153, 159)
    .cartcontrol-wrapper
      position: absolute
      right: 12px
      bottom: 12px
    .buy
      position: absolute
      right: 18px
      bottom: 18px
      z-index: 10
      height: 24px
      line-height: 24px
      padding: 0 12px
      box-sizing: border-box
      font-size: 10px
      border-radius: 12px
      color: #ffffff
      background: rgb(0, 160, 220)
      &.fade-enter-active, &.fade-leave-active
        transition: all 0.2s
      &.fade-enter, &.fade-leave-to
        opacity: 0
  .info
    padding: 18px
    .title
      line-height: 14px
      margin-bottom: 6px
      font-size: 14px
      color: rgb(7, 17, 27)
    .text
      line-height: 24px
      padding: 0 8px
      font-size: 12px
      color: #4d555d
  .rating
    padding-top: 18px
    .title
      line-height: 14px
      margin-left: 18px
      font-size: 14px
      color: rgb(7, 17, 27)
    .rating-wrapper
      padding: 0 18px
      .rating-item
        position: relative
        padding: 16px 0
        border-1px(rgba(7, 17, 27, 0.1))
        .user
          position: absolute
          right: 0px
          top: 16px
          line-height: 12px
          font-size: 0px
          .name
            display: inline-block
            vertical-align: top
            margin-right: 6px
            font-size: 10px
            color: rgb(147, 153, 159)
          .avatar
            border-radius: 50%
        .time
          margin-bottom: 6px
          line-height: 12px
          font-size: 10px
          color: rgb(147, 153, 159)
        .text
          line-height: 16px
          font-size: 12px
          color: rgb(7, 17, 27)
          .icon-thumb_up, .icon-thumb_down
            margin-right: 4px
            line-height: 16px
            font-size: 12px
          .icon-thumb_up
            color: rgb(0, 160, 220)
          .icon-thumb_down
            color: rgb(147, 153, 159)
      .no-rating
        padding: 16px 0
        font-size: 12px
        color: rgb(147, 153, 159)
</style>
