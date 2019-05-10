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
                <a :href="item.linkUrl">
                  <img class="needsclick" :src="item.image" :alt="item.title">
                </a>
              </div>
            </slider>
          </div>
        </div>

        <div class="menu-wrapper" ref="menuWrapper">
            <ul class="menu-content" ref="menuContent">
              <li class="menu-item" ref="menuItem" v-for="(service, index) in serviceMenuList"
                :class="[ index===0  ? 'click-item' : '']"
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

    <div v-show="isShowFiexd" class="menu-wrapper fiexd-menu fade" ref="menuWrapperFiexd">
      <ul class="menu-content" ref="menuContentFiexd">
        <li class="menu-item" ref="menuItem" v-for="(service, index) in serviceMenuList"
          :class="[ index===currentIndex  ? 'click-item' : '']"
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

  export default {
    components: {
      ShopHeader,
      Scroll,
      Slider
    },
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
        isShowFiexd: false
      }
    },
    created () {
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

              this.fartherScroll.on('scroll', (pos) => {

                this.scrollY = Math.abs(pos.y)

                if(this.scrollY >= bannerHieght ){
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
        var top = parseInt(bannerHieght + menuHeight)
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
        this.$router.push({path: data})
      }
    },
    computed: {
      currentIndex () {
        const {tops, scrollY} = this
        // scrollY大于或等于当前的top, 且小于下一个top
        return this.packedIndex = tops.findIndex((top,index) => {
          return scrollY>=top && scrollY<tops[index+1]
        })
      }
    },
    watch: {
      currentIndex(newval){
        var li = this.$refs.menuWrapper.getElementsByClassName('menu-item')[newval - 1]
        var li_fiexd = this.$refs.menuWrapperFiexd.getElementsByClassName('menu-item')[newval - 1]
        this.scroll.scrollToElement(li, 300)
        this.fiexdScroll.scrollToElement(li_fiexd, 300)
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
          color rgb(213,182,94)
          position relative
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
            font-size 1rem
            font-weight 600


    .serviceCenterContent
      position fixed
      left 0
      top 0
      bottom 0
      right 0
      .all-service
        min-height 1px
        // height 100vh
        .banner
          height 9.375rem
          width 100%
          position relative
        .hideBanner
          -webkit-animation slideOutUp 1s
          animation slideOutUp 1s



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


@-webkit-keyframes slideOutUp {
  from {
      -webkit-transform: translateY(0);
      transform: translateY(0)
  }
  to {
    -webkit-transform: translateY(-100%);
    transform: translateY(-100%)
  }
}

@keyframes slideOutUp {
  from {
      -webkit-transform: translateY(0);
      transform: translateY(0)
  }
  to {
    -webkit-transform: translateY(-100%);
    transform: translateY(-100%)
  }
}
</style>
