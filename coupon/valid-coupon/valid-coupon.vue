<template>
  <div class="validCoupon">
    <scroll style="top:3.375rem" ref="validCouponScroll" class="validCouponScroll" :pullup="true"
            @scrollToEnd="loadMore">
      <div>
        <div>
          <coupon-item @handleItem="handleItem" @showDesText="showDesText" v-for="item in couponsList" :key="item.id" :coupondata="item" :valid="valid" class="nav-item"></coupon-item>
          <loading style="padding:  1rem 0" v-show="showLoad" :title="loadingTitle"></loading>
        </div>
        <div v-show="noCoupon" class="no-coupon">
         <div class="no-coupon-content fadeIn">
           <img src="../my-coupon/images/no-coupon.png" alt="no coupon" class="">
           <p class="no-coupon-text">您还没有购买任何卡券</p>
           <button type="button" class="goShop" @click="goShop">前往购买</button>
         </div>
        </div>
      </div>
    </scroll>
  </div>
</template>

<script>
  import Scroll from '../../base/scroll/scroll'
  import ShopHeader from '../../base/shop-header/shop-header'
  import couponItem from '../../base/coupon-item/coupon-item'
  import * as core from '../../api/myCoupon'
  import Loading from '../../base/loading/loading'
  import tool from '../../common/js/util'

  export default {
    name: 'validCoupon',
    data() {
      return {
        valid: true,
        couponsList: [],
        currentPage: 1,
        pageSize: 10,
        showLoad: false,
        noCoupon: false,
        loadingTitle: '正在加载...'
      }
    },
    components: {
      Scroll,
      ShopHeader,
      couponItem,
      Loading
    },
    created() {
      document.title = this.$route.meta.title
      this.getAllCouponsList({
        currentPage: this.currentPage,
        pageSize: this.pageSize
      });
    },
    mounted(){
      if (window.history && window.history.pushState) {
        history.pushState(null, null, document.URL);
        window.addEventListener('popstate', this.goBack, false);
      }
    },
    destroyed(){
      window.removeEventListener('popstate', this.goBack, false);
    },
    methods: {
      goBack () {
        // console.log("点击了浏览器的返回按钮");
        this.$couponToastBox.hideToastBox()
        window.history.back();
      },
      loadMore() {
        if (!this.showLoad && !this.hasMore) {
          this.getAllCouponsList({
            currentPage: this.currentPage,
            pageSize: this.pageSize
          })
        }
      },
      loadImage() {
        this.$refs.validCouponScroll.refresh()
      },
      showDesText(){
        this.$refs.validCouponScroll.refresh()
      },
      handleItem(){
        // console.log("点击了")
      },
      goShop(){
        this.$router.push({path:'/serviceCenter'})
      },
      getAllCouponsList(opts) {
        this.showLoad = true;
        core.getAllCoupon(opts).then(res => {
          //console.log(res);
          if (res.code && '00' === res.code) {
            this.loadingTitle = '';
            this.currentPage++;
            this.showLoad = false;
            this.couponsList = this.couponsList.concat(res.result.data)
            if (this.couponsList.length >= res.result.amount) {
              this.hasMore = true
            }
            if(this.couponsList.length < 1){
              this.noCoupon = true
            }
          }else if (res.code && '01' === res.code) {
            tool.toWeiXinLogin(window.location.href)
          } else {
            this.showLoad = false
            this.$toastBox.showToastBox(res.message)
          }
        })
      }
    }
  }
</script>

<style lang="stylus" rel="stylesheet/stylus" scoped>
  .validCoupon
    .validCouponScroll
      position absolute
      top 0
      left 0
      right 0
      bottom 0
      background-color rgb(245, 245, 245)
      overflow hidden

    .no-coupon
      position fixed
      top 0
      left 0
      right 0
      bottom 0
      .no-coupon-content
        position absolute
        top 6rem
        width 9.5rem
        left 50%
        transform translateX(-50%)
        text-align center
        img
          width 9.5rem
          height 6.78125rem
        .no-coupon-text
          margin 0.75rem 0 0.9375rem 0
          font-size 0.75rem
          color rgb(153,153,153)
        .goShop
          outline none
          background transparent
          color rgb(66,176,233)
          border 0.0625rem solid rgb(66,176,233)
          border-radius 4px
          padding 0.22rem  0.625rem

.fadeIn {
    -webkit-animation: fadeIn .3s;
            animation: fadeIn .3s;
}

@-webkit-keyframes fadeIn {
    from {
      opacity: 0
    }
    to {
      opacity: 1
    }
}

@keyframes fadeIn {
    from {
        opacity: 0
    }
    to {
        opacity: 1
    }
}
</style>
