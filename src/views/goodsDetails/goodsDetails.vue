<template>
  <div class="goods" :style="{height: style.goodsHeight + 'px'}" ref="goods">
    <headerui :title="title" :back="true" ref="headerui"></headerui>

    <div class="goods-index" @touchstart="goodsTouchstart" @touchmove="goodsTouchmove" @touchend="goodsTouchend">
      <div class="goods-img">
        <img :src="imgUrl" v-cloak alt="">
      </div>
      <!-- 商品信息 -->
      <div class="goods-info">
        <div class="goods-name" v-html="goodDetail.name"></div>
        <div class="goods-prices">{{goodDetail.point}} 积分</div>
      </div>
      <!-- 商品数量 -->
      <div class="goods-number">
        <div>数量:</div>
        <div class="add-num" @click.stop="subNumber">-</div>
        <div class="goods-num">{{goodNumber}}</div>
        <div class="sub-num" @click.stop="addNumber">+</div>
      </div>
      <!-- 上滑 拉出图文详情 -->
      <div class="goods-touch" ref="promptText" @touchstart="goodsTouchstart" @touchmove="goodsTouchmove" @touchend="goodsTouchend">
        上拉查看图文详情
      </div>
    </div>

    <!-- 图文详情 -->
    <div :class="{'goods-details': true}" :style="{top: style.detailsTop + 'px'}" ref="details" @touchstart.stop="presentationTouchstart" @touchmove.stop="presentationTouchmove" @touchend.stop="presentationTouchend">
      <mt-navbar v-model="selected" class="nav-box">
        <button :class="{btn: true, active: index == idn}" v-for="(item,index) in navArr" :id="index + ''" @click="clickNav(item,index)">
          <span>{{item}}</span>
        </button>
      </mt-navbar>

      <!-- tab-container -->
      <mt-tab-container v-model="selected" class="nav-item">
        <mt-tab-container-item id="0">
          <div v-html="goodDetail.desc">
          </div>
        </mt-tab-container-item>
        <mt-tab-container-item id="1">
          <div v-html="goodDetail.spec">
          </div>
        </mt-tab-container-item>
        <mt-tab-container-item id="2">
          <div v-html="goodDetail.package">
          </div>
        </mt-tab-container-item>
      </mt-tab-container>
    </div>

    <!-- 子路由页面 -->
    <router-view :goodDetail="goodDetail" :goodNumber="goodNumber"></router-view>

    <footer-pay @click.native='clickFooterPay'></footer-pay>
  </div>
</template>

<script>
import headerui from 'components/headerui/headerui'
import footerPay from 'components/footer-pay/footer-pay'
import { request, requestGet } from 'common/js/request'

export default {
  data() {
    return {
      // 商品详情
      goodDetail: {},
      // 商品数量
      goodNumber: 1,
      // 头部标题
      title: '福励云积分-商品详情',
      // 商品详情控件
      navArr: ['商品介绍', '规格参数', '包装售后'],
      // html样式 控件
      style: {
        // 内容高度
        goodsHeight: 0,
        // 商品详情块top高度
        detailsTop: 0,
        // 判断 可以上拉拉出图文详情模块的条件(大于49才行)
        clientBottom: 0
      },
      // tuch类事件 数据
      goodsTouch: {
        start: 0,
        move: 0,
        end: 0
      },
      presentationTouch: {
        start: 0,
        move: 0,
        end: 0
      },
      // 图文页 滚动top
      scrollTop: 0,
      // nav 选中标签的id (使用btn点击更新selected实现切换)
      selected: '0',
      //   nav 样式控件
      idn: 0
    }
  },
  methods: {
    // 切换 nav
    clickNav(item, index) {
      console.log('切换nav')
      console.log(item, index)
      this.selected = index + ''
      this.idn = index
    },
    // 监听 图文详情页 滚动事件
    handleScroll() {
      console.log('滚动事件 图文详情页')
      var scrollTop =
        this.$refs.details.pageYOffset || this.$refs.details.scrollTop
      console.log(scrollTop)
      this.scrollTop = scrollTop
    },
    // + - 商品数量
    addNumber() {
      console.log('商品数量+1')
      if (this.goodNumber < 10) {
        this.goodNumber += 1
      }
    },
    subNumber() {
      console.log('商品数量-1')
      if (this.goodNumber > 1) {
        this.goodNumber -= 1
      }
    },
    // 底部 立即兑换 按钮
    clickFooterPay() {
      console.log('底部 兑换/确定地址')
      this.$router.push({ name: 'pay' })
    },
    // 商品页 拉动 事件
    goodsTouchstart(e) {
      console.log('goods 拉动 start')
      console.log(e)
      this.goodsTouch.start = e.changedTouches[0].pageY
    },
    goodsTouchmove(e) {
      console.log('goods 拉动 move')
      console.log(e)
      var move = e.changedTouches[0].pageY
      if (this.goodsTouch.start - move > 0) {
        console.log('上拉')
        this.style.detailsTop =
          this.style.goodsHeight - (this.goodsTouch.start - move)
      }
    },
    goodsTouchend(e) {
      console.log('goods 拉动 end')
      console.log(e)
      //   没有拉动80px就返回
      if (this.style.goodsHeight - this.style.detailsTop >= 80) {
        this.style.detailsTop = 0
      } else {
        this.style.detailsTop = this.style.goodsHeight
      }
    },
    // 图文详情页 拉动事件
    presentationTouchstart(e) {
      console.log('presentation 拉动 start')
      console.log(e)
      // 图文详情页没滚动(在顶部)才可以下拉,否则不行
      console.log(this.scrollTop)
      if (this.scrollTop !== 0) return
      this.presentationTouch.start = e.changedTouches[0].pageY
    },
    presentationTouchmove(e) {
      console.log('presentation 拉动 move')
      console.log(e)
      var move = e.changedTouches[0].pageY
      if (move - this.presentationTouch.start > 0) {
        console.log('下拉')
        this.style.detailsTop = move - this.presentationTouch.start
      }
    },
    presentationTouchend(e) {
      console.log('presentation 拉动 end')
      console.log(e)
      //   没有拉动80px就返回
      if (this.style.detailsTop >= 80) {
        this.style.detailsTop = this.style.goodsHeight
      } else {
        this.style.detailsTop = 0
      }
    }
  },
  computed: {
    active(index) {
      return index === this.index
    },
    imgUrl() {
      if (this.goodDetail.pic) {
        return 'http://tlink.cc/po-back/' + this.goodDetail.pic
      } else {
        return null
      }
    }
  },
  mounted() {
    // 请求 商品 详情
    var id = this.$route.params.id
    var url = '/wel/Shop/goods_detail'
    // var data = { id: id }
    requestGet(url, {
      // params: {
      id: id
      // }
    }).then(res => {
      this.goodDetail = res.data
    })

    // 获取手机屏幕高度
    console.log(document.documentElement.clientHeight)
    // 减去上下headerui和tabbar高度
    var topHeight = this.$refs.headerui.$el.clientHeight
    var height = document.documentElement.clientHeight - (topHeight + 50) // 底部footer-pay固定50
    this.style.goodsHeight = height // main 高度 (除了上下)
    this.style.detailsTop = height // details top

    // 添加一个监听滚动事件 叫handleScroll 写在methods里
    this.$refs.details.addEventListener('scroll', this.handleScroll)
  },
  components: {
    headerui,
    footerPay
  },
  // 离开页面要清除handleScroll事件
  beforeDestroy() {
    console.log('销毁 滚动事件')
    this.$refs.details.removeEventListener('scroll', this.handleScroll)
  }
}
</script>

<style scoped lang='less'>
@import url('../../common/less/variable');
[v-cloak] {
  display: none;
}
.goods {
  overflow: hidden;
  margin-top: 1.333333rem;
  position: relative;
  .goods-index {
    width: 100%;
    height: 100%;
    .goods-img {
      width: 100%;
      height: 8rem;
      & img {
        width: 100%;
        height: 100%;
      }
    }
    .goods-info {
      text-align: left;
      padding-left: 0.533333rem;
      padding-right: 0.533333rem;
      margin-top: 0.533333rem;
      margin-bottom: 0.266667rem;
      .goods-name {
        font-size: @font-size-medium-x;
        line-height: 0.533333rem;
        //多于两行隐藏,并显示省略号
        overflow: hidden;
        text-overflow: ellipsis;
        display: -webkit-box; //将元素设为盒子伸缩模型显示
        -webkit-box-orient: vertical; //伸缩方向设为垂直方向
        -webkit-line-clamp: 2; //超出3行隐藏，并显示省略号
      }
      .goods-prices {
        font-size: @font-size-small;
        color: @color-dialog-background;
        margin-top: 0.266667rem;
      }
    }
    .goods-number {
      line-height: 30px;
      padding-left: 20px;
      font-size: 16px;
      display: flex;
      & div:nth-last-child(-n + 3) {
        width: 25px;
      }
      .add-num,
      .sub-num {
        width: 20px;
      }
      .goods-num {
      }
    }
    .goods-touch {
      font-size: @font-size-medium;
      color: @color-dialog-background;
      line-height: 1.066667rem;
      background-color: @color-background-d;
      position: absolute;
      left: 0;
      right: 0;
      bottom: 0;
    }
  }

  // 图文详情页
  .goods-details {
    width: 100%;
    height: 100%;
    position: absolute;
    left: 0;
    right: 0;
    top: 0;
    background-color: @color-background-d;
    overflow: auto;
    .nav-box {
      border-bottom: 1px solid @color-background-d;
      display: flex;
      justify-content: space-around;
      position: absolute;
      left: 0;
      right: 0;
      top: 0;
      & .btn {
        font-size: @font-size-medium;
        color: @color-dialog-background;
        background-color: #fff;
        line-height: 0.533333rem;
        padding-top: 0.266667rem;
        padding-bottom: 0.266667rem;
        outline: none;
        border: none;
        flex: 1;
      }
      & .btn:nth-child(1) span,
      & .btn:nth-child(2) span {
        width: 100%;
        display: inline-block;
        border-right: 1px solid @color-dialog-background;
      }
      // 当前按钮样式
      .active {
        color: @color-theme;
      }
    }
    .nav-item {
      margin-top: 1.066667rem;
      background-color: @color-background-d;
    }
  }
}
</style>


