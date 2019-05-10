<template>
  <div class="myCoupon">
    <shop-header v-if="showHeader" style="position: absolute;top:0;left: 0;z-index: 999" line-style="background:#fff" title="我的卡券"></shop-header>
    <div :style="myCouponStyle" ref="myCouponScroll" class="myCouponScroll">
      <div>
        <div class="coupon-category">
          <div class="coupon-tab coupon-tab-valid">
            <router-link class="coupon-list" :to="{ path: 'validCoupon' }">有效期内</router-link>
          </div>
          <div class="coupon-tab">
            <router-link class="coupon-list" :to="{ path: 'invalidCoupon' }">已失效</router-link>
          </div>
        </div>
        <router-view/>
      </div>
    </div>
  </div>
</template>

<script>
  import Scroll from '../../base/scroll/scroll'
  import ShopHeader from '../../base/shop-header/shop-header'

  export default {
    name: 'myCoupon',
    data() {
      return {
        couponsList: [],
        showHeader: false,
        myCouponStyle: "",
      }
    },
    components: {
      Scroll,
      ShopHeader,
    },
    created() {
      document.title = this.$route.meta.title
      let ua = navigator.userAgent.toLowerCase();
      let isWeixin = ua.indexOf('micromessenger') != -1;

      if (!isWeixin) {
        this.showHeader=true
        this.myCouponStyle = "top:2.75rem"
        // alert('Dragondean说这不是通过微信内置浏览器');
      } else {
        this.showHeader=false
        this.myCouponStyle = "top:0rem"
      }
    },
    methods: {

    }
  }
</script>

<style lang="stylus" rel="stylesheet/stylus" scoped>
  .myCoupon
    .myCouponScroll
      position fixed
      top 0
      left 0
      right 0
      bottom 0
      background-color rgb(245, 245, 245)
      .coupon-category
        width 100%
        height 2.75rem
        line-height 2.75rem
        border-top 1px solid rgb(230,230,230)
        display flex
        justify-content flex-start
        background-color rgb(255,255,255)
        // -moz-box-shadow 0px 1px 2px #c1c1c1
        // -webkit-box-shadow 0px 1px 2px #c1c1c1
        // box-shadow 0px 1px 2px #c1c1c1
        .coupon-tab
          flex 1
          font-size 0.875rem
          text-align center
          color rgb(102,102,102)
          position relative
          .coupon-list
            height 100%
            display inline-block
          .router-link-active
            color rgb(255,72,0)
            border-bottom 3px solid rgb(255,72,0)
            box-sizing: border-box;
        .coupon-tab-valid::after
          content ""
          width 2px
          height 1.7rem
          position absolute
          top 0.525rem
          right -1px
          background: -moz-linear-gradient(top, #fff, #ececec, #ddd, #ececec, #fff);
          background: -webkit-gradient(linear, top, bottom, from(#fff), color-stop(0.25, #ececec), color-stop(0.5, #ddd), color-stop(0.75, #ececec), to(#fff));
          background: -webkit-linear-gradient(top, #fff, #ececec, #ddd, #ececec, #fff);
          background: -o-linear-gradient(top, #fff, #ececec, #ddd, #ececec, #fff);
          background: linear-gradient(#fff, #ececec, #ddd, #ececec, #fff);

</style>
