<template>
  <div class="service-center">
    <shop-header ref="shopHeader" v-if="showHeader" style="position: absolute;top:0;left: 0;z-index: 999" line-style="background:#fff" title="服务中心"></shop-header>

    <div :style="serviceCenterStyle" class="serviceCenterContent" ref="serviceCenterContent">

      <div class="all-service" >

        <div class="banner" ref="bannerCont">
          <div v-if="swiperList.length" class="slider-wrapper"
             ref="sliderWrapper">
            <slider :loop="true" :auto-play="true" :isClick="true">
              <div v-for="(item,index) in swiperList" :key="index">
                <!-- <a  :href="item.linkUrl"> -->
                <a @click="jumpUrl(item.linkUrl)">
                  <img class="needsclick" :src="item.image" :alt="item.title">
                </a>
              </div>
            </slider>
          </div>
        </div>

        <div v-show="!isShowFiexd" class="menu-wrapper fadeOut" ref="menuWrapper">
            <ul class="menu-content" ref="menuContent">
              <li class="menu-item" ref="menuItem" v-for="(service, index) in serviceMenuList"
                :class="[ index===0  ? 'click-item' : 'noClick-item']"
                @click="clickMenuItem(index, $event)">
                <span class="text">
                  {{service.moduleName}}
                </span>
              </li>
            </ul>
        </div>

        <div class="services-Wrapper" ref="servicesWrapper">
          <ul class="services-content" ref="servicesContent">
            <li class="service-list" ref="serviceList" v-for="(service, index) in serviceMenuList">

              <div>
                <h2 v-show="index==0? false : true" class="service-title">{{ service.moduleName }}</h2>
                <div class="service-itemWrap">
                  <div class="service-item " v-for="item in service.dataList" @click="clickServieItem(item.linkUrl,$event)">
                    <div class="icon">
                      <img :src="item.image" :alt="item.title">
                      <div class="service-name">{{ item.title }}</div>
                      <div class="service-des">{{ item.subTitle }}</div>
                    </div>
                  </div>
                </div>
                <div class="listLast" ref="listLast" v-if="index===serviceMenuList.length-1" :style="listLastHeight"></div>
              </div>

            </li>
          </ul>
        </div>

      </div>

    </div>

    <div v-show="isShowFiexd" class="menu-wrapper fiexd-menu fadeIn" ref="menuWrapperFiexd">
      <ul class="menu-content" ref="menuContentFiexd">
        <li class="menu-item" ref="menuItem" v-for="(service, index) in serviceMenuList"
          :class="[ index===currentIndex  ? 'click-item' : 'noClick-item']"
          @click="clickMenuFiexdItem(index, $event)">
          <span class="text">
            {{service.moduleName}}
          </span>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
  import BScroll  from 'better-scroll'
  import ShopHeader from '../../base/shop-header/shop-header'
  import Scroll from '../../base/scroll/scroll'
  import Slider from '../../base/slider/slider'
  import * as core from '../../api/serviceCenter'
  import tool from '../../common/js/util'
  import wxShareMixin from '../../common/js/wxShareMixin'

  export default {
    components: {
      ShopHeader,
      Scroll,
      Slider
    },
    mixins:[wxShareMixin],
    data () {
      return {
        swiperList: [],
        showHeader: false,
        serviceCenterStyle: "",
        servicesTopstyle:"",
        isShowBanner: false,
        serviceMenuList: [],
        tops: [],
        scrollY: 0,
        packedIndex: 0,
        listLastHeight:'',
        isShowFiexd: false,
        shareUrl: location.href.split('#')[0],
        shareLink:  window.location.href.split("#")[0]+'#'+window.location.href.split("#")[1],  //分享出去的链接
        shareTitle: '袋鼠会员特权',  //分享的标题
        shareDesc: '多达100余种会员特权，每年能帮您节省5998元，畅享世界从这里开始！', //分享的详情介绍
        shareImgUrl: 'https://c1.51jujibao.com/static/mkt/2019/05/shareImages/huiyuan.png',
      }
    },
    created () {
      this.getShare()
      document.title = this.$route.meta.title
      let ua = navigator.userAgent.toLowerCase();
      let isWeixin = ua.indexOf('micromessenger') != -1;
      if (!isWeixin) {
        this.showHeader=false
        this.serviceCenterStyle = "top:0rem"
        // alert('Dragondean说这不是通过微信内置浏览器');
      } else {
        this.showHeader=false
        this.serviceCenterStyle = "top:0rem"
      }

      core.getServiceCenter().then(res => {
        this.$nextTick(() => {
          //console.log(res)
          if(res.code && '00' == res.code){
            this.serviceMenuList = res.result.moduleMap.group_service_list
            if(res.result.moduleMap.mod_banner.dataList){
              this.swiperList = res.result.moduleMap.mod_banner.dataList
            }
            this._initScroll()
          }else if (res.code && '01' === res.code) {
            tool.toWeiXinLogin(window.location.href)
          } else {
            this.$toastBox.showToastBox(res.message)
          }
        })
      })

    },
    mounted(){
      this.$nextTick(()=>{

      });
    },
    methods: {
      jumpUrl(e){
          window.top.location=e
      },
      _initScroll () {
        // 创建分类列表的Scroll对象
        this.$nextTick(()=>{
            let width = 0
            for (let i = 0, length=this.serviceMenuList.length; i < length; i++) {
              width+=this.$refs.menuItem[0].getBoundingClientRect().width
            }
            this.$refs.menuContent.style.width = width+'px'

            this.$refs.menuContentFiexd.style.width = width+'px'

            if (!this.scroll) {
              this.fartherScroll = new BScroll(this.$refs.serviceCenterContent,{
                probeType: 3,
                startY: 0,
                scrollX: false,
                scrollY: true,
                bounce: false,
                click: true
              })

              let bannerHieght = this.$refs.bannerCont.offsetHeight
              let menuHieght = this.$refs.menuContent.offsetHeight

              this.fartherScroll.on('scroll', (pos) => {

                this.scrollY = Math.abs(pos.y)

                if(this.scrollY >= bannerHieght){
                  this.serviceCenterStyle = "top:2.75rem"
                  this.isShowFiexd = true
                  this.scroll.disable()
                  this.fiexdScroll.enable()
                }else{
                  this.serviceCenterStyle = "top:0rem"
                  this.isShowFiexd = false
                  this.fiexdScroll.disable()
                  this.scroll.enable()
                }

              })

              this.scroll = new BScroll(this.$refs.menuWrapper, {
                startX: 0,
                scrollX: true,
                scrollY: false,
                probeType: 3,
                click: true
              })

              this.fiexdScroll = new BScroll(this.$refs.menuWrapperFiexd, {
                startX: 0,
                scrollX: true,
                scrollY: false,
                probeType: 3,
                click: true
              })

              // 占位高度
              let length=this.serviceMenuList.length
              let lastHeight=this.$refs.serviceList[length-1].offsetHeight
              let windowHeight = window.innerHeight || document.documentElement.clientHeight || document.body.clientHeight //窗口高度
              let menuHeight = this.$refs.menuWrapper.getBoundingClientRect().height
              let placeholderHeight = windowHeight - lastHeight
              this.listLastHeight =  "height:"+ placeholderHeight + "px"

              //初始化itemHeight
              this._initTops()
            }else{
              this.fartherScroll.refresh()
              this.scroll.refresh()
              this.fiexdScroll.refresh()
            }
        })
      },
      _initTops(){
        let bannerHieght = this.$refs.bannerCont.offsetHeight
        let menuHeight = this.$refs.menuWrapper.getBoundingClientRect().height
        var tops = this.tops
        var top = parseInt(bannerHieght - 1)
        tops.push(top)
        var lis = this.$refs.servicesWrapper.getElementsByClassName('service-list');
        [].slice.call(lis).forEach(li => {
          top += li.clientHeight
          tops.push(top)
        })
      },
      clickMenuItem(index,event){
        // 过滤掉原生DOM事件
        if(!event._constructed) { // _constructed是better-scroll库添加的
          //console.log("nofeixd")
          return
        }
        //console.log("nofiedx",index)

        let bannerHieght = this.$refs.bannerCont.offsetHeight

        //直接点击需要滚动到位置禁用父级滚动
        this.fartherScroll.scrollTo(0, -bannerHieght,1000)

        var li = this.$refs.servicesWrapper.getElementsByClassName('service-list')[index]
        this.fartherScroll.scrollToElement(li, 300)
      },
      clickMenuFiexdItem(index,event){
        // 过滤掉原生DOM事件
        if(!event._constructed) { // _constructed是better-scroll库添加的
         // console.log("feixd")
          return
        }
        //console.log("fiedx",index)
        var li = this.$refs.servicesWrapper.getElementsByClassName('service-list')[index]
        this.fartherScroll.scrollToElement(li, 300)
      },
      clickServieItem(data,event){
        if(!event._constructed) {
          //console.log("1",event)
        }
        //console.log("2",event)
        // console.log(data)
        // window.location.href='/shop/market/v1/index.html#'+data
        window.top.location='/shop/market/v1/index.html#'+data
        // this.$router.push({path: data})
      }
    },
    computed: {
      currentIndex() {
        for (let i = 0; i < this.tops.length; i++) {
          let height1 = this.tops[i];
          let height2 = this.tops[i + 1];
          if (this.scrollY >= height1 && this.scrollY < height2) {
            return i;
          }
        }
        return 0;
      }
    },
    watch: {
      currentIndex(newval){
        var li = this.$refs.menuWrapper.getElementsByClassName('menu-item')[newval - 1]
        var li_fiexd = this.$refs.menuWrapperFiexd.getElementsByClassName('menu-item')[newval - 1]
        this.scroll.scrollToElement(li, 200)
        this.fiexdScroll.scrollToElement(li_fiexd, 200)
      }
    },
    activated(){
      if (!this.fartherScroll) {
        //console.log("meiyou1")
      }else{

      }
    }
  }
</script>


<style lang="stylus" rel="stylesheet/stylus" scoped>
  .service-placeholder
    width 100%
  .service-center
    background-color #fff
    .fiexd-menu
      position fixed
      left 0
      top 0
      z-index 999

    .menu-wrapper
      width 100%
      background #fff
      overflow hidden
      height 2.71875rem
      border-bottom: 1px solid #ddd;
      .menu-content
        height 2.71875rem
        &:first-child
          margin-left
        .menu-item
          width 5.625rem
          height 2.71875rem
          line-height 2.71875rem
          float left
          text-align center
          .text
            font-size 0.875rem
            width 100%
            font-weight 600
        .click-item
          position relative
          // -webkit-animation changeMenu 0.2s
          // animation changeMenu 0.2s
          color rgb(213,182,94)
          &::after
            content ""
            position absolute
            bottom 0
            left 50%
            transform translateX(-50%)
            width 1.875rem
            height 0.1875rem
            background-color rgb(213,182,94)
            border-radius 5px
          .text
            font-weight 600
            font-size 1rem

    .serviceCenterContent
      position fixed
      left 0
      top 0
      bottom 0
      right 0
      max-width 750px
      .all-service
        min-height 1px
        // height 100vh
        .banner
          height 9.375rem
          width 100%
          position relative
          overflow hidden
        .hideBanner
          // -webkit-animation slideOutUp 1s
          // animation slideOutUp 1s

        .services-Wrapper
          overflow hidden
          padding 0 1rem 0rem 1rem
          .service-list
            padding-top 1.0625rem
            &:first-child
              padding-top 0
          .service-title
            font-size 0.937rem
            position relative
            height 1rem
            font-weight 600
            padding-left 0.5rem

            &::before
              content ""
              position absolute
              left 0;
              top 0;
              height 0.875rem
              width 0.1875rem
              background-color rgb(213,182,94)
              border-radius 2px

          .service-itemWrap
            display flex
            flex-wrap wrap
            padding-top 0.75rem

            .service-item
              width 25%
              height 7.375rem
              position relative
              .icon
                text-align center
                img
                  height 3.125rem
                  width 3.125rem
                  border-radius 50%
                  display block
                  margin 0 auto
                  margin-top 0.625rem
                  background-color: #ddd;
                .service-name
                  font-size 0.8125rem
                  margin-top 0.75rem
                .service-des
                  margin-top 0.5rem
                  font-size 0.75rem
                  color: rgb(153,153,153)

.fadeIn
  -webkit-animation fadeIn 0.2s
  animation fadeIn 0.2s

.fadeOut
  -webkit-animation fadeOut 0.2s
  animation fadeOut 0.2s


@-webkit-keyframes changeMenu {
  from {
    font-size 0.5rem
  }
  to {
    font-size 1rem
  }
}

@keyframes changeMenu {
  from {
    font-size 0.5rem
  }
  to {
    font-size 1rem
  }
}
</style>
