<template>
  <view>
    <view class="container">
      <view class="book-info">
        <view class="book-head">
          <image mode="aspectFill" :src="detail.books.pic" alt="" />
          <view class="title">{{detail.books.title || '加载中..'}}</view>
          <view class="author"> {{detail.books.author || '加载中..'}}</view>
        </view>
      </view>
    </view>
    <view class="container" style="margin-bottom: 180rpx">
      <view class="summary">
        <view style="border-left: 10rpx solid #1296db;width: 20prx;margin-right: 20rpx;"></view>
        本书简介
      </view>
      <view class="summary" style="padding-top: 0">
        {{detail.books.summary || '加载中..'}}
      </view>
    </view>
    <view class="btn" v-if="userInfo.id && (detail.books.user_id === userInfo.id) && detail.books.share_status === 'UNSHARED'" hover-class="click-hover" @tap="changeStatus('SHARED', detail.books.id)">共享图书</view>
    <view class="btn" v-if="userInfo.id && (detail.books.user_id === userInfo.id) && detail.books.share_status === 'SHARED'" hover-class="click-hover" @tap="changeStatus('UNSHARED', detail.books.id)">取消共享图书</view>
    <view class="btn" v-if="userInfo.id && (detail.books.user_id !== userInfo.id) && detail.books.share_status === 'SHARED'" hover-class="click-hover" @tap="changeStatus('borrow', detail.books.id)">我要借书</view>
    <button class="btn" hover-class="click-hover"
    v-if="!userInfo.id"
    open-type="getUserInfo"
    @getuserinfo="bindGetUserInfo"
    withCredentials="true"
    lang="zh_CN"
    >我要借书</button>
  </view>
</template>

<script>

export default {
  data() {
    return {
      detail: {},
      userInfo: {}
    }
  },
  async onLoad(option) {
    console.log(option)
    this.userInfo = uni.getStorageSync('loginInfo')
    this.detail = await this.$api.post('/api2/books/shareBooks/getBookDetail', {
      isbn: option.isbn,
      userId: this.userInfo.id,
      shareStatus: option.shareStatus
    })
    console.log(this.detail)
  },
  methods: {
    async changeStatus(shareStatus, bookId) {
      if (shareStatus === 'borrow') return this.$api.goPage(`./borrowBook?bookId=${bookId}&shareStatus=${shareStatus}`)
      let {message} = await this.$api.post('/api2/books/shareBooks/shareBookStatus', {bookId, shareStatus})
      this.detail.books.share_status = shareStatus
      this.$api.showModal(message)
    },
    async bindGetUserInfo(e) {
      console.log(e)
      if (e.detail.errMsg !== 'getUserInfo:ok') return
      if (!this.$_.isEmpty(this.userInfo)) return
      await this.$wxService.auth(this.$globalData.appid, e.detail)
      this.userInfo = await this.$wxService.login()
    },
  }

}
</script>

<style lang='scss'>
page {
  background: #f6f6f6;
}
.container {
  background: #fff;
  padding: 0;
}
.book-head {
  text-align: center;
  margin-bottom: 30rpx;
  padding-top: 30rpx;
  padding-bottom: 30rpx;
  image {
    width: 160px;
    height: 400rpx;
  }
  .title {
    font-size: 36rpx;
    margin-top: 20rpx;
    margin-bottom: 20rpx;
  }
  .author {
    font-size: 28rpx;
    color: #606764;
  }
}
.summary {
  display: flex;
  padding: 30rpx;
  font-size: 30rpx;
}
.btn {
  background: #1296db;
  height: 90rpx;
  position: fixed;
  bottom: 0rpx;
  width: 100%;
  text-align: center;
  line-height: 100rpx;
  color: #fff;
  font-size: 36rpx;
}
button {
  border: none;
  outline: none;
  padding: 0;
  border-radius: 0
}
button:after {border: none; clear: both;}

</style>
