<template>
  <div>
    <div class="goods">

      <div class="menu-wrapper" ref="menuWrapper" >
        <ul>
          <li v-for="(item, index) in goods" class="menu-item" :class="{'current':currentIndex === index}" 
          v-bind:key="index" @click="selectMenu(index, $event)">
          <span class="text border-1px">
            <GoodsTypeIcon :size="3" :type="item.type" class="icon"></GoodsTypeIcon>
            <!--<span v-show="item.type>0" class="icon" :class="classMap[item.type]"></span>-->
            {{item.name}}
          </span>
          </li>
        </ul>
      </div>
      <div class="foods-wrapper" ref="foodWrapper">
        <ul>
          <li v-for="(item, index) in goods" class="food-list food-list-hook" v-bind:key="index">
            <h1 class="title">{{item.name}}</h1>
            <ul>
              <li v-for="(food, index) in item.foods" class="food-item" @click="selectFood(food, $event)"
              v-bind:key="index">
                <div class="icon">
                  <img :src="food.icon" alt="" width="57px" height="57px">
                </div>
                <div class="content">
                  <h2 class="name">{{food.name}}</h2>
                  <p class="desc">{{food.description}}</p>
                  <div class="extra">
                    <span class="count">月售{{food.sellCount}}份</span><span>好评率{{food.rating}}%</span>
                  </div>
                  <div class="price">
                    <span class="now">￥{{food.price}}</span>
                    <span v-show="food.oldPrice" class="old">￥{{food.oldPrice}}</span>
                  </div>
                  <div class="cartcontrol-wrapper">
                    <cartcontrol :food="food" :eventHub="eventHub"></cartcontrol>
                  </div>
                </div>
              </li>
            </ul>
          </li>
        </ul>
      </div>
      <shopcart :select-foods="selectFoods" :delivery-price="seller.deliveryPrice" :min-price="seller.minPrice"
                :eventHub="eventHub"></shopcart>
    </div>
    <food :food="selectedFood" ref="foodxq" :eventHub="eventHub"></food>
  </div>
</template>

<script type="text/ecmascript-6">
  import GoodsTypeIcon from '../goodsTypeIcon/goodsTypeIcon.vue';
  import BScroll from 'better-scroll';
  import shopcart from '../shopcart/shopcart.vue';
  import cartcontrol from '../cartcontrol/cartcontrol.vue';
  import food from '../food/food.vue';
  import Vue from 'vue';
  export default {
    name: 'goods',
    props: {
      seller: {
          type: Object
      }
    },
    data () {
      return {
        goods: [],
        listHeight: [],
        scrollY: 0,
        selectedFood: {},
        eventHub: new Vue()
      };
    },
    created() {
      this.classMap = ['decrease', 'discount', 'special', 'invoice', 'guarantee'];
      //  promise链式调用
      this.$http.get('/api/goods').then((response) => {
        return response.json();
      }).then((json) => {
        this.goods = json.goods;
        //  dom操作置后
        this.$nextTick(() => {
          this._initScroll();
          this._calculateHeight();
        });
      });
    },
    methods: {
      _initScroll() {
        this.menuScroll = new BScroll(this.$refs.menuWrapper, {
            click: true
        });
        this.foodScroll = new BScroll(this.$refs.foodWrapper, {
            click: true,
            probeType: 3
        });// 检测滚动位置
        this.foodScroll.on('scroll', (pos) => {
            this.scrollY = Math.abs(Math.round(pos.y));
        });
      },
      _calculateHeight() {
        let foodList = this.$refs.foodWrapper.getElementsByClassName('food-list-hook');
        let height = 0;
        this.listHeight.push(height);
        for (let i = 0; i < foodList.length; i++) {
            let item = foodList[i];
            height += item.clientHeight;
            this.listHeight.push(height);
        }
      },
      selectMenu(index, event) {
          //  原生的点击事件
        if (!event._constructed) {
            return;
        }
        let foodList = this.$refs.foodWrapper.getElementsByClassName('food-list-hook');
        let el = foodList[index];
        this.foodScroll.scrollToElement(el, 300);
      },
      selectFood(food, event) {
          if (!event._constructed) {
              return;
          };
          this.selectedFood = food;
          this.$refs.foodxq.show();// 调用子组件的方法
      }
    },
    computed: {
      currentIndex() {
        //  实时跟踪变化
        for (let i = 0; i < this.listHeight.length; i++) {
          let height1 = this.listHeight[i];
          let height2 = this.listHeight[i + 1];
          if (!height2 || this.scrollY >= height1 && this.scrollY < height2) {
            return i;
          }
        }
        return 0;
      },
      selectFoods() {
         let foods = [];
         this.goods.forEach(good => {
             good.foods.forEach(food => {
                 if (food.count) {
                     foods.push(food);
                 }
             });
         });
         return foods;
      }
    },
    components: {
        GoodsTypeIcon,
        shopcart,
        cartcontrol,
        food
    }
  };
</script>
<style lang="stylus" rel="stylesheet/stylus" type="text/stylus">
  @import "../../common/stylus/mixin.styl"
.goods
  position: absolute
  width: 100%
  top: 174px
  bottom: 46px
  display: flex
  overflow: hidden
  .menu-wrapper
    flex: 0 0 80px
    width: 80px
    background: #f3f5f7
    .menu-item
      display: table
      height: 54px
      width: 56px
      line-height: 14px
      padding: 0 12px
      &.current
        position: relative
        z-index: 10
        margin-top: -1px
        background: #fff
        font-weight: 700
        .text
          border-none()
      .icon
        width: 12px
        height: 12px
        margin-right: 2px
        background-size: 12px 12px
      .text
        font-size: 12px
        display: table-cell
        width: 56px
        height: 54px
        vertical-align: middle
        border-1px(rgba(7, 17, 27, 0.1))
  .foods-wrapper
    flex: 1
    .title
      padding-left: 14px
      height: 26px
      line-height: 26px
      border-left: 2px solid #d9dde1
      font-size: 12px
      color: rgb(147, 153, 159)
      background: #f3f5f7
    .food-item
      display: flex
      margin: 18px
      padding-bottom: 18px
      border-1px(rgba(7, 17, 27, 0.1))
      &:last-child
        border-none()
        margin-bottom: 0
      .icon
        flex: 0 0 57px
        margin-right: 10px
      .content
        flex: 1
        .name
          margin: 2px 0 8px 0
          height: 14px
          line-height: 14px
          font-size: 14px
          color: rgb(7, 17 ,27)
        .desc, .extra
          line-height: 10px
          font-size: 10px
          color: rgb(147, 153 ,159)
        .desc
          line-height: 12px
          margin-bottom: 8px
        .extra
          .count
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
          right: 0
          bottom: 12px
</style>
