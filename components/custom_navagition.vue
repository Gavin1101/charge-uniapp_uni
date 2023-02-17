<template>
  <view>
    <!-- 自定义头部导航 -->
    <view class="custom_navagition" :style="{backgroundColor: backgroundColor}">
      <view class="booth" :style="{height: stateHeight + 'px',color: color}"></view>
      <view class="navagition" :style="{height: navHeight + 'px',lineHeight: navHeight + 'px',color: color,fontWeight: fontWeight,fontSize: fontSize}"><text>{{tit}}</text></view>
      <slot></slot>
    </view>
  </view>
</template>

<script>
  export default {
    name:"custom-navagition",
    props: {
      bgc: String,
      title: String,
      col: String,
      fw: String,
      fs: String
    },
    data() {
      return {
        navHeight: 0,
        top: 0,
        stateHeight: 0,
        
        tit: '',
        backgroundColor: '',
        color: '',
        fontWeight: '',
        fontSize: ''
      };
    },
    created() {
      this.tit = this.$props.title
      this.backgroundColor = this.$props.bgc
      this.color = this.$props.col
      this.fontWeight = this.$props.fw
      this.fontSize = this.$props.fs
      this.loadNavagition()
    },
    methods: {
      // 自定义导航栏高度兼容
      loadNavagition() {
        this.stateHeight = 0
        this.top = 0
        let that = this
        uni.getSystemInfo({
          success(res) {
            that.stateHeight = res.statusBarHeight;
          }
        })
        this.top = uni.getMenuButtonBoundingClientRect().top - this.stateHeight
        this.navHeight = uni.getMenuButtonBoundingClientRect().height + (this.top * 2)
      },
    }
  }
</script>

<style lang="scss">
  .custom_navagition {
    position: sticky;
    top: 0;
    z-index: 1;

    .booth {
    }

    .navagition {
      text-align: center;
      font-size: 36rpx;
      font-weight: 600;
      color: #000000;
    }
  }  
</style>