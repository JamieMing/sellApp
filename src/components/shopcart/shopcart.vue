<template>
  <div>
    <div class="shopcart">
      <div class="content" @click="toggleList">
        <div class="content-left">
          <div class="logo-wrapper">
            <div class="logo" :class="{'highlight':totalCount>0}">
              <span class="icon-shopping_cart" :class="{'highlight':totalCount>0}"></span>
            </div>
            <div class="num" v-show="totalCount>0">{{totalCount}}</div>
          </div>
          <div class="price" :class="{'highlight':totalPrice>0}">￥{{totalPrice}}元</div>
          <div class="desc">另需配送费{{deliveryPrice}}元</div>
        </div>
        <div class="content-right" @click.stop.prevent="pay">
          <div class="pay" :class="payClass">
            {{payDesc}}
          </div>
        </div>
      </div>
      <div class="ball-container">
        <template v-for="ball in balls">
          <transition
            v-on:before-enter="beforeEnter"
            v-on:enter="enter"
            v-on:after-enter="afterEnter"
            name="drop">
            <div  v-show="ball.show" class="ball">
              <div class="inner inner-hook"></div>
            </div>
          </transition>
        </template>

      </div>
      <transition name="fold">
        <div class="shopcart-list" v-show="listShow">
          <div class="list-header">
            <h1 class="title">购物车</h1>
            <span class="empty" @click="empty">清空</span>
          </div>
          <div class="list-content" ref="listContent">
            <ul>
              <li class="food" v-for="food in selectFoods">
                <span class="name">{{food.name}}</span>
                <div class="price">
                  <span>￥{{food.price*food.count}}</span>
                </div>
                <div class="cartcontrol-wrapper">
                  <cartcontrol :food="food"></cartcontrol>
                </div>
              </li>
            </ul>
          </div>
        </div>
      </transition>
    </div>
    <transition name="fade">
      <div class="list-mask" v-show="listShow" @click="hideList"></div>
    </transition>
  </div>
</template>

<script type="text/ecmascript-6">
  import cartcontrol from '../cartcontrol/cartcontrol.vue';
  import BScroll from 'better-scroll';
  export default {
    props: {
        selectFoods: {
          type: Array,
          default() {
              return [];
          }
        },
        deliveryPrice: {
            type: Number,
            default: 0
        },
        minPrice: {
            type: Number,
            default: 0
        },
        eventHub: {
          type: Object
        }
    },
    data() {
      return {
          balls: [
            {
                show: false
            },
            {
              show: false
            },
            {
              show: false
            },
            {
              show: false
            },
            {
              show: false
            }
          ],
          dropBalls: [],
          fold: true
      };
    },
    computed: {
        totalPrice() {
            let total = 0;
            this.selectFoods.forEach(food => {
                total += food.price * food.count;
            });
          return total;
        },
        totalCount() {
            let count = 0;
            this.selectFoods.forEach(food => {
                count += food.count;
            });
            return count;
      },
        payDesc() {
          if (this.totalPrice === 0) {
            return `￥${this.minPrice}元起送`;
          } else if (this.totalPrice < this.minPrice) {
            let diff = this.minPrice - this.totalPrice;
            return `还差${diff}元起送`;
          } else {
              return '去结算';
          }
        },
        payClass() {
            return this.totalPrice < this.minPrice ? 'not-enough' : 'enough';
      },
        listShow() {
            if (!this.totalCount) {
                this.fold = true;
                return false;
            }
            let show = !this.fold;
            if (show) {
                this.$nextTick(() => {
                    if (!this.scroll) {
                      this.scroll = new BScroll(this.$refs.listContent, {
                        click: true
                      });
                    } else {
                        this.scroll.refresh();
                    }
                });
            }
            return show;
        }
    },
    created() {
        this.eventHub.$on('cart.add', (target) => {
            this.$nextTick(() => {
              this._drop(target);
            });
        });
    },
    methods: {
        _drop(el) {
            for (let i = 0; i < this.balls.length; i++) {
              let ball = this.balls[i];
              if (!ball.show) {
                ball.show = true;
                ball.el = el;
                this.dropBalls.push(ball);
                return;
              }
          }
        },
        beforeEnter(el) {
          let count = this.balls.length;
          while (count--) {
              let ball = this.balls[count];
              if (ball.show) {
                  let rect = ball.el.getBoundingClientRect();
                  let x = rect.left - 32;
                  let y = -(window.innerHeight - rect.top - 22);
                  el.style.display = '';
                  el.style.webkitTransform = `translate3d(0, ${y}px,0)`;
                  el.style.transform = `translate3d(0, ${y}px,0)`;
                  let inner = el.getElementsByClassName('inner-hook')[0];
                  inner.style.webkitTransform = `translate3d(${x}px,0,0)`;
                  inner.style.transform = `translate3d(${x}px,0,0)`;
              }
          }
        },
        enter(el) {
            /* eslint-disable no-unused-vars */
              let rf = el.offsetHeight;
              el.style.webkitTransform = 'translate3d(0, 0,0)';
              el.style.transform = 'translate3d(0, 0,0)';
              let inner = el.getElementsByClassName('inner-hook')[0];
              inner.style.webkitTransform = 'translate3d(0, 0,0)';
              inner.style.transform = 'translate3d(0, 0,0)';
              // done();
        },
        afterEnter(el) {
              let ball = this.dropBalls.shift();
              if (ball) {
                ball.show = false;
                el.style.display = 'none';
              }
        },
        toggleList() {
            if (!this.totalCount) {
                return;
            }
            this.fold = !this.fold;
        },
        hideList() {
          this.fold = true;
        },
        empty() {
            this.selectFoods.forEach((food) => {
                food.count = 0;
            });
        },
        pay() {
            if (this.totalPrice < this.minPrice) {
                return;
            }
            window.alert(`支付${this.totalPrice}`);
        }
    },
    components: {
        cartcontrol: cartcontrol
    }
  };
</script>
<style lang="stylus" rel="stylesheet/stylus" type="text/stylus">
  @import "../../common/stylus/mixin.styl"
.shopcart
  position: fixed
  left: 0
  bottom: 0
  z-index: 50
  width: 100%
  height: 48px
  background: #000
  .content
    display: flex
    background: #141d27
    font-size: 0
    .content-left
      flex: 1
      .logo-wrapper
        display: inline-block
        position: relative
        top: -10px
        margin: 0 12px
        padding: 6px
        width: 56px
        height: 56px
        box-sizing: border-box
        vertical-align: top
        border-radius: 50%
        background: #141d27
        .logo
          width: 100%
          height: 100%
          border-radius: 50%
          background: #2b343c
          text-align: center
          &.highlight
            background: rgb(0, 160, 220)
          .icon-shopping_cart
            line-height: 44px
            font-size: 24px
            color: #80858a
            &.highlight
              color: #fff
        .num
          position: absolute
          top: 0
          right: 0
          width: 24px
          height: 16px
          line-height: 16px
          text-align: center
          border-radius: 16px
          font-size: 9px
          font-weight: 700px
          background: rgb(240, 20, 20)
          color: #fff
          box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.4)
      .price
        display: inline-block
        vertical-align: top
        margin-top: 12px
        line-height: 24px
        padding-right: 12px
        box-sizing: border-box
        border-right: 1px solid rgba(255, 255, 255, 0.1)
        font-size: 16px
        font-weight: 700
        color: rgba(255, 255, 255, 0.4)
        &.highlight
          color: #fff
      .desc
        display: inline-block
        vertical-align: top
        margin: 12px 0 0 12px
        line-height: 24px
        color: rgba(255, 255, 255, 0.4)
        font-size: 10px
    .content-right
      flex: 0 0 105px
      width: 105px
      .pay
        height: 48px
        line-height: 48px
        text-align: center
        font-size: 12px
        color: rgba(255, 255, 255, 0.4)
        font-weight: 700

        &.not-enough
          background: #2b333b
        &.enough
          background: #00b43c
          color: #fff
  .ball-container
    .ball
      position: fixed
      left: 32px
      bottom: 22px
      transition: all .4s cubic-bezier(0.49, -0.29, 0.75, 0.41)
      .inner
        width: 16px
        height: 16px
        border-radius: 50%
        background: rgb(0, 160, 220)
        transition: all .4s linear
  .shopcart-list
    position: absolute
    top: 0
    left: 0
    z-index: -1
    width: 100%
    transform: translate3d(0, -100%, 0)
    &.fold-enter-active, &.fold-leave-active
      transition: all .5s
    &.fold-enter, &.fold-leave-to
      transform: translate3d(0, 0, 0)
    .list-header
      height: 40px
      line-height: 40px
      padding: 0 18px
      background: #f3f5f7
      border-bottem: 1px solid rgba(7, 17, 27, 0.1)
      .title
        float: left
        font-size: 14px
        color: rgb(7, 17, 27)
      .empty
        float: right
        font-size: 12px
        color: rgb(0, 160, 220)
    .list-content
      padding: 0 18px
      max-height: 217px
      overflow: hidden
      background: #fff
      .food
        position: relative
        padding: 12px 0
        box-sizing: border-box
        border-1px(rgba(7, 17, 27, 0.1))
        .name
          line-height: 24px
          font-size: 14px
          color: rgb(7, 17, 27)
        .price
          position: absolute
          right: 98px
          bottom: 12px
          line-height: 24px
          color: rgb(240, 20, 20)
          font-weight: 700px
          font-size: 14px
        .cartcontrol-wrapper
          position: absolute
          right: 0
          bottom: 6px
.list-mask
  position: fixed
  top: 0
  left: 0px
  width: 100%
  height: 100%
  z-index: 40
  background: rgba(7, 17, 27, 0.6)
  -webkit-backdrop-filter: blur(10px)
  &.fade-enter-active, &.fade-leave-active
    transition: all .5s
  &.fade-enter, &.fade-leave-to
    opacity: 0
    background: rgba(7, 17, 27, 0)
</style>
