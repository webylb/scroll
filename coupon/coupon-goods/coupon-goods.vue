<template>
  <div class="coupon-goods">
    <shop-header ref="shopHeader" v-if="showHeader" style="position: absolute;top:0;left: 0;z-index: 999" line-style="background:#fff" :title="title"></shop-header>

    <div :style="couponGoodsStyle" class="couponGoodsContent" ref="couponGoodsContent">
      <div class="goods-detail">
        <div class="banner" ref="bannerCont">
          <img src="./images/goodsbanner.png" alt="" class="banner-bg">
          <div class="banner-cont">
            <div class="banner-ico">
              <img :src="bannerImg" :alt="title">
              <div class="banner-title">{{ title }}</div>
            </div>
            <div class="banner-btn">
              <button type="button" class="btn" @click="goMyCoupon">我的卡券<i class="iconfont">&#xe624;</i></button>
            </div>
          </div>
        </div>
        <div class="goods-content" v-if="couponData">

          <div class="goods-content-item">
            <div class="goods-type-title">充值方式</div>
            <div class="goods-type-way">
              <button type="button" v-for="(recharge,index) in couponData" :key="index" class="btn" :class="[ rechargeType === index ? 'btn-active' : 'btn-default']" @click="changeRecharge(index)">{{ recharge.title }}</button>
            </div>
          </div>
          <div v-for="(recharge,index) in couponData" :key="index" class="goodsData">
            <div v-if="index === rechargeType">

              <div v-if="recharge.rechargeAccountTypeList" class="goods-content-item">
                <div class="goods-type-title">充值账号</div>
                <div class="goods-type-way">
                  <div class="input-wrap" v-for="(rechargeInputItem,index) in recharge.rechargeAccountTypeList">
                    <input type="text" v-model='inputData' ref="rechargeInputItem" :name="rechargeInputItem.keyName" :placeholder="rechargeInputItem.placeHolder" @click.stop="focusInput"  @blur="scrollToTop" maxlength="50" class="recharge-input" />
                  </div>
                </div>
              </div>

              <div class="goods-content-item">
                <div class="goods-type-title">类型</div>
                <div class="goods-type-way">
                  <button type="button" v-for="(genre,index) in recharge.subList" :key="index" class="btn" :class="[ genreType === index ? 'btn-active' : 'btn-default']" @click="changeGenre(index)">{{ genre.title }}</button>
                </div>
              </div>

              <div v-for="(genre,index) in recharge.subList" ::key="index">
                <div v-if="index === genreType">
                  <div class="goods-content-item">
                    <div class="goods-type-title">面额</div>
                    <div class="goods-type-way goods-type-wrap">
                      <div v-for="(goods,index) in recharge.subList[index].subList" :key="index" class="goods" :class="[ goodsType === index ? 'goods-active' : 'goods-default']" @click="changeGood(index,goods.sku)" ref="goods" :default-sku="goods.sku">
                        <div class="goods-img-wrap">
                          <div class="goods-title">{{ goods.title }}</div>
                          <div class="good-price-wrap">
                            <div class="goods-price"><span>￥</span>{{ goods.price }}</div>
                            <div class="goods-oldPrice"><s>市场价:￥{{ goods.oldPrice }}</s></div>
                            <div class="goods-id">{{ goods.id }}</div>
                          </div>
                        </div>
                        <div v-if="goods.cornerMarker" class="goods-mark">
                          <img :src="goods.cornerMarker" alt="">
                        </div>
                      </div>
                      <div class="goods placeholder-goods"></div>
                      <div class="goods placeholder-goods"></div>
                    </div>
                  </div>
                </div>
              </div>

              <div class="goods-content-item">
                <div class="goods-type-title">使用说明</div>
                <div v-html="recharge.content" class="goods-type-way goods-directionForUse">
                    <!-- <div v-html="recharge.content">{{ recharge.content }}</div> -->
                    {{ recharge.content }}
                </div>
              </div>

            </div>
          </div>

        </div>
      </div>
    </div>
    <div class="immediate-payment" @click="immediatePay">
      立即付款
    </div>
    <addvip-toast :showAddvip="showAddvip" :vipTypeList="vipTypeList" :vipTypeDefaultId="vipTypeDefaultId" @closeVipBox="closeVipBox" @addvip="addvip"></addvip-toast>
  </div>
</template>

<script>
  import BScroll  from 'better-scroll'
  import ShopHeader from '../../base/shop-header/shop-header'
  import Scroll from '../../base/scroll/scroll'
  import addvipToast from '../../base/addvip-toast/addvip-toast'
  import * as core from '../../api/serviceCenter'
  import tool from '../../common/js/util'
  import wxShareMixin from '../../common/js/wxShareMixin'


  export default {
    components: {
      ShopHeader,
      Scroll,
      addvipToast
    },
    mixins:[wxShareMixin],
    data () {
      return {
        showHeader: false,
        title: "",
        couponGoodsStyle: "",
        bannerImg: "",
        rechargeType: 0,
        rechargeAccount: '',
        genreType: 0,
        goodsType: 0,
        goodsSku: "",
        couponData: {},
        inputData: '',
        showAddvip: false,
        isVipUser: false,
        vipTypeList: [],
        vipTypeDefaultId: 0,
        isPaying: true,
        shareUrl: location.href.split('#')[0],
        shareLink:  window.location.href.split("#")[0]+'#'+window.location.href.split("#")[1],  //分享出去的链接
        shareTitle: '',  //分享的标题
        shareDesc: '', //分享的详情介绍
        shareImgUrl: '',
      }
    },
    created () {
      document.title = this.$route.meta.title
      //判断是否为微信
      let ua = navigator.userAgent.toLowerCase();
      let isWeixin = ua.indexOf('micromessenger') != -1;
      if (!isWeixin) {
        this.showHeader=true
        this.couponGoodsStyle = "top:2.75rem"
      } else {
        this.showHeader=false
        this.couponGoodsStyle = "top:0rem"
      }

      core.getServiceCenterDetail({pageKey: this.$route.query.pageKey}).then(res => {
        //console.log(res)
        if(res.code && '00' == res.code){

          // shareUrl: location.href.split('#')[0],
          //   shareLink:  window.location.href.split("#")[0]+'#'+window.location.href.split("#")[1],  //分享出去的链接
          //   shareTitle: '',  //分享的标题
          //   shareDesc: '', //分享的详情介绍
          //   shareImgUrl: '',

          this.shareTitle=res.result.pageName+'特权'
          this.shareDesc='积分袋鼠'+res.result.pageName+'特权，每年能帮您节省5998元，畅享世界从这里开始！'
          this.shareImgUrl=res.result.moduleMap.mod_banner.dataList[0].image
          this.getShare();

          this.couponData = res.result.moduleMap.mod_item_list.dataList
          this.title = res.result.pageName
          if(res.result.moduleMap.mod_banner.dataList){
            this.bannerImg = res.result.moduleMap.mod_banner.dataList[0].image
          }
          this.$nextTick(() => {
            this._initScroll()
            this.changeTitle()
            this.checkDefaultSku()
          })
        }else if (res.code && '01' === res.code) {
          tool.toWeiXinLogin(window.location.href)
        } else {
          this.$toastBox.showToastBox(res.message)
        }
      })
    },
    mounted(){
      this.$nextTick(()=>{

      });
    },
    watch: {
      rechargeType(oldval,val){
        this.$nextTick(()=>{
          this.checkDefaultSku();
          //console.log(this.goodsSku)
        })
      }
    },
    methods: {
      focusInput(e){
        e.target.focus()
      },
      scrollToTop(){
        //console.log("blur")
        window.scrollBy(0,10)
      },
      changeTitle(){
        let head = document.getElementsByTagName('head');
        let meta = document.createElement('meta');
        head[0].appendChild(meta)
        document.title = this.title;
      },
      checkDefaultSku(){
        this.goodsSku = this.$refs.goods[0].getAttribute("default-sku")
      },
      _initScroll () {
        // 创建分类列表的Scroll对象
        this.$nextTick(()=>{
            if (!this.scroll) {
              this.scroll = new BScroll(this.$refs.couponGoodsContent,{
                probeType: 3,
                startY: 0,
                bounce: false,
                click: true
              })
            }else{
              this.scroll.refresh()
            }
        })
      },
      goMyCoupon() {
        this.$router.push("/validCoupon")
      },
      //卡密充值
      changeRecharge(val){
        this.rechargeType = val

        this.genreType = 0
        this.goodsType = 0
      },
      //类型
      changeGenre(val){
        this.genreType = val

        if(this.$refs.rechargeInputItem){
          this.$refs.rechargeInputItem[0].blur()
        }
      },
      changeGood(val,sku){
        this.goodsType = val
        this.goodsSku = sku

        if(this.$refs.rechargeInputItem){
          this.$refs.rechargeInputItem[0].blur()
        }
      },
      immediatePay(){
        // 判断用户是否为vip
        this.isVip()
      },
      isVip(){
        core.isVip().then(res => {
          //console.log(res)
          if(res.code && '00' == res.code){
            this.isVipUser = res.result.isVipUser
            if(this.isVipUser){
              if(this.isPaying){
                this.isPaying = false
                //console.log(this.isPaying)
                this.toPlay()
              }
            }else{
              this.vipTypeList = res.result.vipTypeList
              this.vipTypeDefaultId = res.result.vipTypeList[0].id
              this.showAddvip = true
            }
          } else if (res.code && '01' === res.code) {
            tool.toWeiXinLogin(window.location.href)
          } else {
            this.$toastBox.showToastBox(res.message)
          }
        })
      },
      addvip(data){
        let returnUrl = window.location.href
        core.addVip({vipTypeId: data,returnUrl:returnUrl}).then(res => {
          //console.log(res)
          if(res.code && '00' == res.code){
            window.location.href = res.result.payUrl
          }else if (res.code && '01' === res.code) {
            tool.toWeiXinLogin(window.location.href)
          } else {
            this.$toastBox.showToastBox(res.message)
          }
        })
      },
      toPlay(){
        let data = {}
        if(this.$refs.rechargeInputItem && this.$refs.rechargeInputItem.length > 0){
          if(this.inputData){
            data = {sku: this.goodsSku,account:this.inputData}
          }else{
            this.$toastBox.showToastBox("请输入充值账号!")
            return;
          }
        } else {
          data = {sku: this.goodsSku}
        }
        core.buyCoupon(data).then(res => {
          //console.log(res)
          if(res.code && '00' == res.code){
            window.location.href = res.result.payUrl
          }else if (res.code && '01' === res.code) {
            tool.toWeiXinLogin(window.location.href)
          } else {
            this.$toastBox.showToastBox(res.message)
            this.isPaying = true
          }
        })
      },
      closeVipBox(opt){
        this.showAddvip = opt
      }
    },
    updated() {

    },
  }
</script>


<style lang="stylus" rel="stylesheet/stylus" scoped>
  @import "../../common/stylus/iconfont.styl"
  .coupon-goods
    background-color #fff
    .immediate-payment
      position fixed
      bottom 0
      right 0
      width 100%
      height 3.0625rem
      line-height 3.0625rem
      background-color rgb(235,215,155)
      color rgb(111,65,9)
      font-size 1.125rem
      text-align center
      font-weight: 600;
    .couponGoodsContent
      position fixed
      left 0
      top 0
      bottom 0
      right 0
      .goods-detail
        min-height 1px
        .banner
          width 100%
          height 4.6875rem
          overflow hidden
          position relative
          .banner-bg
              width 100%
              height 100%
              display inline-block
          .banner-cont
            position absolute
            left 1rem
            right 1rem
            top 0.9375rem
            bottom 0
            background-color rgba(37,37,37,0.9)
            border-top-left-radius 5px
            border-top-right-radius 5px
            display flex
            justify-content space-between
            align-items center
            padding 0 1rem
            overflow hidden
            &::after
              position absolute
              content ''
              right -1rem
              bottom -1rem
              width 2rem
              height 2rem
              background rgba(235,215,155,0.2)
              border-radius 50%
            .banner-ico
              display flex
              align-items center
              img
                max-height 1.9375rem
                width auto
                margin-right 0.3125rem
              .banner-title
                font-size 1.0625rem
                color rgb(235,215,155)
                font-weight 600
            .banner-btn
              // width 5rem
              height 1.5rem
              line-height 1.5rem
              button
                width 100%
                height 100%
                background-color rgb(235,215,155)
                color rgb(37,37,37)
                border none
                border-radius 0.75rem
                outline none
                padding 0 0.5rem
                i
                  font-size 0.5rem

        .goods-content
          padding 1.25rem 1rem
          .goods-content-item
            margin-bottom 2.21875rem
            .goods-type-title
              font-size 0.9375rem
              font-weight 600
            .goods-type-way
              margin-top 0.9375rem
              font-size: 0;
              .input-wrap + .input-wrap
                margin-top 0.75rem
              .btn
                min-width 5rem
                outline none
                font-size 0.9375rem
                border-radius 4px
                box-sizing border-box
                color rgb(111,65,9)
                height 2.5rem
                line-height 2.5rem
                padding 0 0.5625rem
              .btn+.btn
                margin-left 0.625rem
              .btn-default
                background-color rgb(255,254,250)
                border 1px solid rgb(243,231,195)
              .btn-active
                background-color rgb(255,250,234)
                border 1px solid rgb(205,172,78)
              .recharge-input
                -webkit-user-select : auto
                width 100%
                outline none
                font-size 0.9375rem
                border-radius 4px
                box-sizing border-box
                color rgb(111,65,9)
                height 2.5rem
                line-height 2.5rem
                padding 0 0.5625rem
                background-color rgb(255,254,250)
                border 1px solid rgb(205,172,78)

              .goods
                width 6.75rem
                font-size 0.9375rem
                box-sizing border-box
                color rgb(111,65,9)
                // height 5.9375rem
                text-align center
                position relative
                margin-bottom 1.25rem
                .goods-mark
                  width 2.75rem
                  height 0.875rem
                  position absolute
                  right 0
                  top -0.375rem
                  img
                    width 100%
                    height 100%
                    display inline-block
                .goods-img-wrap
                  overflow hidden
                .goods-title
                  font-size 0.9375rem
                  height 2.125rem
                  line-height 2.125rem
                  position relative
                  border-top-left-radius 4px
                  border-top-right-radius 4px
                  &::before
                    position absolute
                    content ""
                    width 0.875rem
                    height 0.875rem
                    left calc(-0.875rem/2)
                    bottom calc(-0.875rem/2)
                    border-radius 50%
                    background-color #fff
                    border 1px solid rgb(243,231,195)
                  &::after
                    position absolute
                    content ""
                    width 0.875rem
                    height 0.875rem
                    right calc(-0.875rem/2)
                    bottom calc(-0.875rem/2)
                    border-radius 50%
                    background-color #fff
                    border 1px solid rgb(243,231,195)
                .good-price-wrap
                  height 4rem
                  border-bottom-left-radius 4px
                  border-bottom-right-radius 4px
                  padding-top 0.84rem
                  box-sizing: border-box;
                  .goods-price
                    font-size 1.25rem
                    margin-bottom 0.625rem
                    font-weight 600
                    span
                      font-size 0.6875rem
                      font-weight 600
                  .goods-oldPrice
                    font-size 0.6875rem
                    margin-bottom 0.8125rem
                  .goods-id
                    display none
              .goods-default
                background-color rgb(255,254,250)
                .goods-title
                  border 1px solid rgb(243,231,195)
                  border-bottom none
                .good-price-wrap
                  border 1px solid rgb(243,231,195)
                  border-top 1px dashed rgb(243,231,195)

              .goods-active
                background-color rgb(255,250,234)
                .goods-title
                  border 1px solid rgb(205,172,78)
                  border-bottom none
                  &::before
                    border 1px solid rgb(205,172,78)
                  &::after
                    border 1px solid rgb(205,172,78)
                .good-price-wrap
                  border 1px solid rgb(205,172,78)
                  border-top 1px dashed rgb(205,172,78)
              .placeholder-goods
                height 0
                margin 0
            .goods-type-wrap
              display flex
              justify-content space-between
              flex-direction row
              flex-wrap wrap
            .goods-directionForUse
              font-size 0.75rem
              text-align justify
              text-justify newspaper
              word-break normal
              overflow hidden
              line-height 1.1rem
              p
                text-align justify
                text-justify newspaper
                word-break normal
                overflow hidden
              span
                text-align justify
                text-justify newspaper
                word-break normal
                overflow hidden
// 改变placeholder
::-webkit-input-placeholder { /* WebKit, Blink, Edge */
  color:  rgb(215,194,169);
}
:-moz-placeholder { /* Mozilla Firefox 4 to 18 */
  color:  rgb(215,194,169);
}
::-moz-placeholder { /* Mozilla Firefox 19+ */
  color:  rgb(215,194,169);
}
:-ms-input-placeholder { /* Internet Explorer 10-11 */
  color:  rgb(215,194,169);
}

// 改变后来生成html样式
.goods-directionForUse >>> p + p
  margin-top 0.375rem
</style>
