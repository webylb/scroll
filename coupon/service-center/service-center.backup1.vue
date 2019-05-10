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
                :class="[ index===currentIndex  ? 'click-item' : '']"
                @click="clickMenuItem(index, $event)">
                <span class="text">
                  {{service.moduleName}}
                </span>
              </li>
            </ul>
        </div>

        <div class="services-Wrapper" ref="servicesWrapper" :style="servicesTopstyle">
          <ul class="services-content" ref="servicesContent">
            <li class="service-list" ref="serviceList" v-for="(service, index) in serviceMenuList" @click="clickServieTitle(index, $event)">

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
        pickIndex: 0,
        tops: [],
        scrollY: 0,
        listLastHeight:''
      }
    },
    created () {
      document.title = this.$route.meta.title
      let ua = navigator.userAgent.toLowerCase();
      let isWeixin = ua.indexOf('micromessenger') != -1;
      if (!isWeixin) {
        this.showHeader=true
        this.serviceCenterStyle = "top:2.75rem"
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
        this.getComponentHeight()
      });
    },
    methods: {
      getComponentHeight(e){
          let windowHeight = window.innerHeight || document.documentElement.clientHeight || document.body.clientHeight //窗口高度
          let header = document.querySelectorAll("div[class='j-header']")[0];
          let headerHeight
          if(header){
            headerHeight = header.clientHeight
          }else{
            headerHeight =  0
          }

          let menuHeight = this.$refs.menuWrapper.getBoundingClientRect().height

          let height = windowHeight - headerHeight - menuHeight
          this.servicesTopstyle = "height:"+ height + "px"
      },
      _initScroll () {
        // 创建分类列表的Scroll对象
        this.$nextTick(()=>{
            let width = 0
            for (let i = 0, length=this.serviceMenuList.length; i < length; i++) {
              width+=this.$refs.menuItem[0].getBoundingClientRect().width
            }
            this.$refs.menuContent.style.width = width+'px'
            if (!this.scroll) {

              this.fartherScroll = new BScroll(this.$refs.serviceCenterContent,{
                probeType: 3,
                startY: 0,
                scrollX: false,
                scrollY: true,
                bounce: false,
                click: false
              })


              let bannerHieght = this.$refs.bannerCont.offsetHeight

              this.fartherScroll.on('scroll', (pos) => {

                let scrollY = Math.abs(pos.y)

                this.servicesScroll.disable()

                if(scrollY == bannerHieght ){
                  this.fartherScroll.disable()
                  this.servicesScroll.enable()
                }

              })

              this.fartherScroll.on('scrollEnd', (ops) => {
                setTimeout(() => {
                  this.servicesScroll.enable()
                }, 1)
              })

              this.scroll = new BScroll(this.$refs.menuWrapper, {
                startX: 0,
                scrollX: true,
                scrollY: false,
                probeType: 3,
                click: true
              })


              let length=this.serviceMenuList.length
              let lastHeight=this.$refs.serviceList[length-1].offsetHeight
              let windowHeight = window.innerHeight || document.documentElement.clientHeight || document.body.clientHeight //窗口高度
              let menuHeight = this.$refs.menuWrapper.getBoundingClientRect().height
              let placeholderHeight = windowHeight - lastHeight - menuHeight
              this.listLastHeight =  "height:"+ placeholderHeight + "px"

              this.servicesScroll = new BScroll(this.$refs.servicesWrapper, {
                probeType: 3,
                scrollX: false,
                scrollY: true,
                bounce: false,
                click: true,
                tap: true,
                //mouseWheel: true
                //stopPropagation: true
              })

              this.servicesScroll.on('scroll', (pos) => {
                this.scrollY = Math.abs(pos.y)
                //console.log(this.scrollY)
                this.fartherScroll.disable()
                if(this.scrollY <= 1){
                  this.fartherScroll.enable()
                  this.servicesScroll.disable()
                }
              })

              //初始化itemHeight
              this._initTops()
            }else{
              this.fartherScroll.refresh()
              this.scroll.refresh()
              this.servicesScroll.refresh()
            }
        })
      },
      _initTops(){
        var tops = this.tops
        var top = 0
        tops.push(top)
        var lis = this.$refs.servicesWrapper.getElementsByClassName('service-list');
        [].slice.call(lis).forEach(li => {
          top += li.clientHeight
          tops.push(top)
        })
        //console.log(tops)
      },
      clickMenuItem(index,event){
        // 过滤掉原生DOM事件
        if(!event._constructed) { // _constructed是better-scroll库添加的
          return
        }

        //console.log(index,event)

        let bannerHieght = this.$refs.bannerCont.offsetHeight

        //直接点击需要滚动到位置禁用父级滚动
        this.fartherScroll.scrollTo(0, -bannerHieght,1000)
        this.fartherScroll.disable()
        this.servicesScroll.enable()

        // this.pickIndex = index
        // this.currentIndex = index
        var li = this.$refs.servicesWrapper.getElementsByClassName('service-list')[index]
        this.servicesScroll.scrollToElement(li, 300)
      },
      clickServieTitle(index,event) {
        if(!event._constructed) {
          return
        }
      },
      clickServieItem(data,event){
        if(!event._constructed) {
          console.log("1",event)
        }
        console.log("2",event)
        this.$router.push({path: data})
      }
    },
    computed: {
      currentIndex: {
        get: function () {
          const {tops, scrollY} = this
          // scrollY大于或等于当前的top, 且小于下一个top
          return tops.findIndex((top,index) => {
            return scrollY>=top && scrollY<tops[index+1]
          })
        },
        set: function (newVal) {
          return newVal
        }
      }
    },
    watch: {
      currentIndex(newval){
        var li = this.$refs.menuWrapper.getElementsByClassName('menu-item')[newval - 1]
        this.scroll.scrollToElement(li, 300)
      }
      // scrollY(val) {
      //   this.pickIndex = this.currentIndex
      // },
      // pickIndex(newval,oldval){
      //   //console.log(newval,oldval)
      //   var li = this.$refs.menuWrapper.getElementsByClassName('menu-item')[newval]
      //   this.scroll.scrollToElement(li, 300)
      // }
    }
  }
</script>


<style lang="stylus" rel="stylesheet/stylus" scoped>
  .service-placeholder
    width 100%
  .service-center
    background-color #fff
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
