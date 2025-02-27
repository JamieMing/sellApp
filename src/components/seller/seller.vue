<template>
  <div class="seller" ref="seller">
    <div class="seller-content">
      <div class="overview">
        <h1 class="title">{{seller.name}}</h1>
        <div class="desc border-1px">
          <star :size="36" :score="seller.score" class="star"></star>
          <span class="text">({{seller.ratingCount}})</span>
          <span class="text">月售{{seller.sellCount}}单</span>
        </div>
        <ul class="remark">
          <li class="block">
            <h2>起送价</h2>
            <div class="content">
              <span class="stress">{{seller.minPrice}}</span>元
            </div>
          </li>
          <li class="block">
            <h2>商家配送</h2>
            <div class="content">
              <span class="stress">{{seller.deliveryPrice}}</span>元
            </div>
          </li>
          <li class="block">
            <h2>平均配送时间</h2>
            <div class="content">
              <span class="stress">{{seller.deliveryTime}}</span>分钟
            </div>
          </li>
        </ul>
        <div class="favorite" @click="toggleFavorite($event)">
          <span class="icon-favorite" :class="{'active': favorite}"></span>
          <span class="text">{{favoriteText}}</span>
        </div>
      </div>
      <split></split>
      <div class="bulletin">
        <h1 class="title">公告与活动</h1>
        <div class="content-wrapper border-1px">
          <p class="content">{{seller.bulletin}}</p>
        </div>
        <ul v-if="seller.supports" class="supports">
          <li class="support-item border-1px" v-for="item in seller.supports">
            <goodsTypeIcon :size="3" :type="item.type" class="icon"></goodsTypeIcon>
            <span class="text">{{item.description}}</span>
          </li>
        </ul>
      </div>
      <split></split>
      <div class="pics">
        <h1 class="title">商家实景</h1>
        <div class="pic-wrapper" ref="picWrapper">
            <ul class="pic-list" ref="picList">
              <li class="pic-item" v-for="pic in seller.pics">
                <img :src="pic" alt="" height="90" width="120">
              </li>
            </ul>
        </div>
      </div>
      <split></split>
      <div class="info">
        <h1 class="title border-1px">商家信息</h1>
        <ul>
          <li class="info-item" v-for="info in seller.infos">
            {{info}}
          </li>
        </ul>
      </div>
    </div>
  </div>

</template>

<script type="text/ecmascript-6">
  import star from '../star/star.vue';
  import split from '../split/split.vue';
  import goodsTypeIcon from '../goodsTypeIcon/goodsTypeIcon.vue';
  import BScroll from 'better-scroll';
  import {saveToLocal, loadFromLocal} from '../../common/js/store';
  export default {
    name: 'seller',
    props: {
      seller: {
          type: Object
      }
    },
    data() {
      return {
          favorite: (() => {
              console.log(loadFromLocal(this.seller.id, 'favorite', false));
              return loadFromLocal(this.seller.id, 'favorite', false);
          })()
      };
    },
    mounted() {
      this.$nextTick(() => {
        this._initScroll();
        this._initPics();
      });
    },
    updated() {
      this.$nextTick(() => {
         this._initScroll();
         this._initPics();
      });
    },
    computed: {
      favoriteText() {
          return this.favorite ? '已收藏' : '收藏';
      }
    },
    methods: {
        _initScroll() {
            if (!this.scroll) {
              this.scroll = new BScroll(this.$refs.seller, {
                click: true
              });
            } else {
                this.scroll.refresh();
            }
        },
        _initPics() {
          if (this.seller.pics) {
            let picWidth = 120;
            let margin = 6;
            let width = (picWidth + margin) * this.seller.pics.length - margin;
            this.$refs.picList.style.width = width + 'px';
            this.$nextTick(() => {
              this.picScroll = new BScroll(this.$refs.picWrapper, {
                scrollX: true,
                eventPassthrough: 'vertical'
              });
            });
          }
        },
        toggleFavorite(event) {
            if (!event._constructed) {
                return;
            }
            this.favorite = !this.favorite;
            saveToLocal(this.seller.id, 'favorite', this.favorite);
        }
    },
    components: {
        star,
        split,
        goodsTypeIcon
    }
  };
</script>
<style lang="stylus" rel="stylesheet/stylus" type="text/stylus">
  @import "../../common/stylus/mixin.styl"
.seller
  position: absolute
  top: 174px
  bottom: 0px
  left: 0px
  width: 100%
  overflow: hidden
  .overview
    position: relative
    padding: 18px
    .title
      margin-bottom: 8px
      line-height: 14px
      font-size: 14px
      color: rgb(7, 17, 27)
    .desc
      font-size: 0
      padding-bottom: 18px
      border-1px(rgba(7, 17, 27, 0.1))
      .star
        display: inline-block
        margin-right: 8px
        vertical-align: top
      .text
        display: inline-block
        margin-right: 12px
        line-height: 18px
        font-size: 10px
        color: rgb(77, 85, 93)
    .remark
      display: flex
      padding-top: 18px
      .block
        flex: 1
        text-align: center
        border-right: 1px solid rgba(7, 17, 27, 0.1)
        &:last-child
          border: none
        >h2
          margin-bottom: 4px
          line-height: 10px
          font-size: 10px
          color: rgb(147, 153, 159)
        .content
          font-size: 10px
          color: rgb(7, 17, 27)
          .stress
            font-size: 24px
            font-weight: 200
    .favorite
      position: absolute
      top: 18px
      right: 14px
      width: 36px
      text-align: center
      >span
        display: block
      .icon-favorite
        margin-bottom: 4px
        font-size: 24px
        line-height: 24px
        color: #d4d6d9
        &.active
          color: rgb(240, 20, 20)
      .text
        line-height: 10px
        font-size: 10px
        color: rgb(77, 85, 93)
  .bulletin
    padding: 18px 18px 0 18px
    .title
      margin-bottom: 8px
      line-height: 14px
      font-size: 14px
      color: rgb(7, 17, 27)
    .content-wrapper
      padding: 0px 12px 16px
      border-1px(rgba(7, 17, 27, 0.1))
      .content
        line-height: 24px
        font-weight: 200
        font-size: 12px
        color: rgb(240, 20, 20)

    .supports
      padding: 0 12px
      .support-item
        font-size: 0px
        padding: 16px 0
        border-1px(rgba(7, 17, 27, 0.1))
        &:last-child
          border-none()
        .icon
          width: 16px
          height: 16px
          margin-right: 6px
          background-size: 16px 16px
        .text
          line-height: 16px
          font-size: 12px
          font-weight: 200
          color: rgb(7, 17, 27)
  .pics
    padding: 18px
    .title
      margin-bottom: 12px
      line-height: 14px
      font-size: 14px
      color: rgb(7, 17, 27)
    .pic-wrapper
      width: 100%
      overflow: hidden
      white-space: nowrap
      .pic-list
        font-size: 0
        .pic-item
          display: inline-block
          margin-right: 6px
          width: 120px
          height: 90px
          &:last-child
            margin: 0
  .info
    padding: 18px 18px 0 18px
    color: rgb(7, 17, 27)
    .title
      margin-bottom: 12px
      line-height: 14px
      font-size: 14px
      border-1px(rgba(7, 17, 27, 0.1))
    .info-item
      padding: 16px 12px
      line-height: 16px
      font-size: 12px
      font-weight: 200
      border-1px(rgba(7, 17, 27, 0.1))
      &:last-child
        border-none()
</style>
