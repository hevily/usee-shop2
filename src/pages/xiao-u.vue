<template>
  <div class="l-page">
    <header class="mui-bar mui-bar-nav l-black" v-if="!$device.isWechat">
      <h1 class="mui-title">{{ $route.meta.title }}</h1>
      <a class="mui-icon mui-icon-arrowleft mui-pull-left _nav-back"></a>
    </header>
    <div class="mui-content">
      <div class="l-loading-inline" v-if="loading" style="padding: 1.8rem 0;"><i class="mui-spinner"></i></div>
      <template v-if="xiaoUInfo.agentId === 1">
        <div class="l-card-count l-text-center">
          <p class="l-fs-s">账户余额(元)</p>  
          <p style="font-size:2rem; margin: 0.5rem 0;">{{xiaoUInfo.readyAmount | currency}}</p>
          <p><button style="background: transparent; border-color: #fff; min-width: 40%;" class="mui-btn l-btn-main _m" @click="$link('/me/account', 'page-in')">查看</button></p>
        </div>
        <div class="l-flex-hc l-bg-white l-text-center" style="padding: 1rem 0;">
          <div class="l-rest" @click="$link('/me/rebate', 'page-in')">
            <p class="l-fs-m">累计返利(元)</p>
            <p class="l-fs-xl">{{xiaoUInfo.amount || 0}}</p>
          </div>
          <div class="l-rest" @click="$link('/me/customer', 'page-in')">
            <p class="l-fs-m">客户数量(人)</p>
            <p class="l-fs-xl">{{xiaoUInfo.customerNum || 0}}</p>
          </div>
        </div>
        <ul class="mui-table-view mui-table-view-chevron l-margin-t">
          <li class="mui-table-view-cell" @click="$link('/me/qrcode', 'page-in')"> 
            <span class="mui-navigate-right">我的二维码</span>
          </li>
          <li class="mui-table-view-cell" @click="$link('/news', 'page-in')">
            <span class="mui-navigate-right">文章资讯</span>
          </li>
          <li class="mui-table-view-cell" @click="$link('/me/xiaou/rule', 'page-in')">
            <span class="mui-navigate-right">规则说明</span>
          </li>
          <li class="mui-table-view-cell" @click="notify">
            <span class="mui-navigate-right">消息推送</span>
            <div class="mui-switch" :class="{'mui-active': isNotice}">
              <div class="mui-switch-handle"></div>
            </div>
          </li>
        </ul>
        <div class="l-margin l-fs-xs l-text-gray" style="margin-top:0.25rem;">开启消息推送您将收到客户关注及购买返利等微信消息通知</div>
      </template>
      <not-u v-else-if="xiaoUInfo.agentId === 0 || xiaoUInfo.agentId === 2"></not-u>
    </div>
  </div>
</template>
<script>
import notU from 'components/not-u'

let now = new Date()
export default {
  components: {
    notU
  },
  data () {
    return {
      isNotice: false,
      loading: false,
      xiaoUInfo: {
        agentId: '',
        notify: 1,
        amount: 0,
        customerNum: 0,
        goodsNumber: 0,
      }
    }
  },
  methods: {
    notify() {
      this.isNotice = !this.isNotice
      this.$mui.showWaiting()
      this.$server.user.notify(Number(this.isNotice)).then(({data})=>{
        if(!data){
          this.$mui.confirm('接收消息通知需要绑定微信，请授权同步微信信息', '开启通知失败', ['取消', '绑定'], (e)=>{
            if(e.index == 1){
              this.refreshUserInfo()
            }
          })
          this.isNotice = !this.isNotice  
        }
      }).catch(()=>{
        this.isNotice = !this.isNotice
      }).finally(()=>{
        this.$mui.hideWaiting()
      })
    },
    getXiaoUInfo() {
      this.loading = true
      this.$server.getXiaoUInfo().then(({data})=>{
        this.isNotice = data.notify == 0 ? false : true
        this.xiaoUInfo = Object.assign(this.xiaoUInfo, data)
      }).finally(()=>{
        this.loading = false
      })
    }
  },
  created() {
    this.getXiaoUInfo()
  }
}
</script>
<style scoped lang="less">
.l-card-count{
  padding: 1.5rem 0; color: #fff;
  background: url('~assets/images/xiao-u-bg.jpg') no-repeat 50% 50%;
  background-size: 110% 110%;
}

</style>