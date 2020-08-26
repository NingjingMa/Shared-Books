<template>
  <view class="container">
    <view class="con-header">
			<image src="../static/search-icon.png" mode="aspectFill" class="search" />
      <view class="inputBox">
        <input class="input" placeholder="请输入书名、作者" @click = "jumpSearch()"/>
      </view>
      <button class="scan" hover-class="click-hover"
				v-if="!userInfo.id"
        open-type="getUserInfo"
        @getuserinfo="bindGetUserInfo"
        withCredentials="true"
        lang="zh_CN"
			>
				<image src="../static/scan-icon.png" mode="aspectFill" />
			</button>
      <button class="scan" hover-class="click-hover" v-if="userInfo.id" @tap="scan">
				<image src="../static/scan-icon.png" mode="aspectFill" />
			</button>
    </view>
		<view class="share-text" v-if="shareBooks.length">
			<h2>共享图书</h2>
			<view v-if="shareBooks.length === 3" class="link" hover-class="click-hover" @tap="$wxApi.goPage('./shareBooks')">查看全部</view>
		</view>
    <view class="con-body">
			<view v-if="!shareBooks.length" class="no-data">没有更多数据了</view>
      <view class="book" hover-class="click-hover"
			v-for="(item,index) in shareBooks" :key="index"
			@tap="goDetail(item.isbn, 'SHARED')"
			>
        <view class="img-box">
          <image class="book-img" mode="aspectFill" :src="item.pic" alt="" />
        </view>
        <view class="book-text">{{item.title}}</view>
      </view>
    </view>
		<view class="my-books">
			<view class="book-list" hover-class="click-hover"
			v-for="(item,index) in unShareBooks" :key="index"
			@tap="goDetail(item.isbn, 'UNSHARED')"
			>
				<image mode="aspectFill" :src="item.pic" alt="" />
				<view style="padding-top:20rpx; flex:0.7">
					<view class="title">{{item.title}}</view>
					<view class="author">{{item.author}}</view>
					<view class="summary">{{item.summary}}</view>
				</view>
			</view>
		</view>

		<!--
		uni app的文档，跳转到搜索页，当然也可以用source的那个app然后修改。点击search的时候跳转到search.vue 然后点击搜索的里面跳转到书的详情界面，让search可以搜索出内容。-->
  </view>
</template>

<script>

import * as _ from 'underscore'

export default {
  data() {
    return {
      userInfo: {},
      shareBooks: [],
      unShareBooks: [],
      onload: 0,
      search: ''
    }
  },
  methods: {
    async goDetail(isbn, shareStatus) {
      return this.$api.goPage(`./bookDetail?isbn=${isbn}&shareStatus=${shareStatus}`)
    },
    jumpSearch() {
      return this.$api.goPage(`../pages/search`)
    },
    async bindGetUserInfo(e) {
      if (e.detail.errMsg !== 'getUserInfo:ok') return
      if (!this.$_.isEmpty(this.userInfo)) return
      await this.$wxService.auth(this.$globalData.appid, e.detail)
      this.userInfo = await this.$wxService.login()
      this.unShareBooks = await this.getBookList({userId: this.userInfo.id, shareStatus: 'UNSHARED'})
    },
    async getBookList({userId, shareStatus, limit}) {
	  const data = await this.$api.post('/api2/books/shareBooks/getBookList', {
	    search: this.search.trim(),
	    shareStatus,
	    limit,
        userId
	  })
	  return data
    },
    async scan() {
      const [err, data] = await uni.scanCode({scanType: ['barCode']})
      if (err) return
      const res = await this.$api.post('/api2/books/shareBooks/getInfoByIsbn', {
        userId: this.userInfo.id,
        isbn: data.result,
        loading: '正在获取图书信息...'
      })
      return this.goDetail(data.result, res.share_status)
    },
    /* handleSearch: _.debounce(async function(e) {
      if (!this.userInfo.id) return
      const { value } = e.target
      this.search = value
      this.unShareBooks = await this.getBookList({userId: this.userInfo.id, shareStatus: 'UNSHARED'})
    }, 500)*/
  },
  async onLoad() {
    this.userInfo = uni.getStorageSync('loginInfo')
    this.shareBooks = await this.getBookList({shareStatus: 'SHARED', limit: 3})
    this.unShareBooks = this.userInfo.id ? await this.getBookList({userId: this.userInfo.id, shareStatus: 'UNSHARED'}) : []
    this.onload = 1
  },
  async onShow() {
    this.userInfo = uni.getStorageSync('loginInfo')
    if (this.onload === 1) {
      this.unShareBooks = this.userInfo.id ? await this.getBookList({userId: this.userInfo.id, shareStatus: 'UNSHARED'}) : []
      this.shareBooks = await this.getBookList({shareStatus: 'SHARED', limit: 3})
    }
  },
}
</script>

<style lang='scss'>
.container {
  background: #fff;
  padding: 0;
  .con-header {
    display: flex;
    background: #eee;
    height: 90rpx;
    width: 100%;
    padding: 16rpx 14rpx;
		box-sizing: border-box;
		.input{height:100%}
    .inputBox {
      background: #fff;
      width: 600rpx;
      height: 58rpx;
      line-height: 58rpx;
      border-radius: 8rpx;
      padding-left: 58rpx;
      font-size: 28rpx;
      color: #666;
    }
    .scan {
			margin-left: 10rpx;
			image {width: 56rpx;height: 56rpx;}
			border: 0;
			outline: none;
			padding: 0;
			background: none;
		}
		.search {
			width: 35rpx;
			height: 35rpx;
			position: absolute;
			top: 25rpx;
			left: 25rpx;
		}
	}

  .con-body {
    display: flex;
    flex-wrap: wrap;
    padding: 30rpx;
    .book {
      width: 33.33%;
      height: 260rpx;
      padding: 0 0 0 15rpx;
      box-sizing: border-box;
      margin-bottom: 70rpx;
      .img-box {
        .book-img {
          width: 200rpx;
          height: 260rpx;
        }
      }
      .book-text {
        font-size: 24rpx;
        width: 200rpx;
        color: #000;
        display: -webkit-box;
        -webkit-line-clamp: 2;
        text-overflow: ellipsis;
        -webkit-box-orient: vertical;
        overflow: hidden;
      }
    }
	}

	.share-text {
		padding: 30rpx 40rpx 0rpx 40rpx;
		display: flex;
		justify-content: space-between;
		font-size: 36rpx;
		.link {
			color: #1296db;
			font-size: 25rpx;
			margin-top: 12rpx;
		}
		h2 {font-weight: 500}
	}

	.my-books {
		padding: 30rpx;
		.book-list {
			display: flex;
			justify-content: flex-start;
			margin-bottom: 20rpx;
			image {
				width: 150rpx;
				height: 200rpx;
				margin-right: 25rpx;
				flex: 0.3
			}
			.title {
				font-size: 30rpx;
				font-weight: 500;
				margin-bottom: 10rpx;
			}
			.author {
				font-size: 28rpx;
				margin-bottom: 10rpx;
				color: #606764;
			}
			.summary {
				font-size: 25rpx;
				color: #b6b5ba;
				display: -webkit-box;
				text-overflow:ellipsis;
				-webkit-box-orient: vertical;
				-webkit-line-clamp: 2;
				overflow: hidden;
				flex: 0.7
			}
		}
	}
}

.no-data {
	font-size: 25rpx;
	text-align: center;
	width: 100%;
	color: #606764;
}

button:after {border: none; clear: both;}

</style>
