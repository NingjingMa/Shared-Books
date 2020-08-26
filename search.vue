<template>
	<view class="container">
		<view class="tui-searchbox">
			<view class="tui-search-input">
				<icon type="search" :size='13' color='#333'></icon>
				<input confirm-type="search" placeholder="大家都在搜：2019退役球星" :focus="true" auto-focus placeholder-class="tui-input-plholder"
				 class="tui-input" v-model.trim="key" @input="inputKey" @change="handleSearch" maxlength="20"/>
				<icon type="clear" :size='13' color='#bcbcbc' @tap="cleanKey" v-show="key"></icon>
			</view>
			<view class="tui-cancle" @tap="back">取消</view>
		</view>

		<view class="tui-search-history" v-show="history.length>0 && !key">
			<view class="tui-history-header">
				<view class="tui-search-title">搜索历史</view>
				<s-icon name="delete" :size='14' color='#333' @tap="openActionSheet" class="tui-icon-delete"></s-icon>
			</view>
			<view class="tui-history-content">
				<block v-for="(item,index) in history" :key="index">
					<s-tag margin="0 24rpx 24rpx 0" type="gray" shape="circle">{{item}}</s-tag>
				</block>
			</view>
		</view>
		<view v-show="key">
			<view class="tui-header">
				<view class="tui-header-left tui-noboredr">搜索 “{{key}}”</view>
			</view>
			<view class="my-books">
					<view class="book-list" hover-class="click-hover"
					v-for="item in searchList" :key="item.isbn"
					@tap="goDetail(item.isbn, 'SHARED')"
					>

							<image mode="aspectFill" :src="item.pic" alt="" />
							<view style="padding-top:20rpx; flex:0.7">
								<view class="title">{{item.title}}</view>
								<view class="author">{{item.author}}</view>
								<view class="summary">{{item.summary}}</view>
							</view>

				</view>
			</view>

			<view class="tui-result-box">
				<block v-for="(item,index) in searchList" :key="index">

					<view class="tui-result-item" hover-class="tui-opcity" :hover-stay-time="150">
						<rich-text :nodes="item.showKey"></rich-text>
					</view>
				</block>
			</view>
		</view>

		<view class="tui-search-hot">
			<view class="tui-hot-header">
				<view class="tui-search-title">大家正在搜</view>
			</view>
			<view class="tui-hot-content">
				<block v-for="(item,index) in hot" :key="index">
					<tui-tag type="gray" shape="circle">{{item}}</tui-tag>
				</block>
			</view>
		</view>
		<s-actionsheet :show="showActionSheet" :tips="tips" @click="itemClick" @cancel="closeActionSheet"></s-actionsheet>

	</view>

</template>

<script>
import util from '../utils/util'
import * as _ from 'underscore'

export default {
  data() {
    return {
      history: [
        "美洲杯",
        "D站观点",
        "C罗",
        "早安D站",
        "2019退役球星",
        "女神大会",
        "德利赫特",
        "托雷斯",
        "自热火锅",
        "华为手机",
        "有机酸奶"
      ],
      hot: [
        "德利赫特",
        "托雷斯",
        "早安D站",
        "D站观点",
        "德利赫特",
        "美洲杯",
        "华为手机",
        "C罗",
        "自热火锅",
        "2019退役球星",
        "女神大会"
      ],
      key: "",
      showActionSheet: false,
      tips: "确认清空搜索历史吗？",
      searchResult: ["按照展示的列表输入关键词看效果", "thorui", "2019退役球星", "搜索关键词高亮显示", "模拟搜索结果集", "开源不易，需要鼓励", "人人为我，我为人人"],
      searchList: []
    }
  },
  methods: {

    back: function() {
      uni.navigateBack()
    },
    cleanKey: function() {
      this.key = ''
    },
    closeActionSheet: function() {
      this.showActionSheet = false
    },
    openActionSheet: function() {
      this.showActionSheet = true
    },
    itemClick: function(e) {
      let index = e.index
      if (index === 0) {
        this.showActionSheet = false
        this.history = []
      }
    },
    async goDetail(isbn, shareStatus) {
	  return this.$api.goPage(`./bookDetail?isbn=${isbn}&shareStatus=${shareStatus}`)
    },
    handleSearch: _.debounce(async function(e) {
	  const { value } = e.target
	  console.log(this)
	  this.key = value
	  this.searchList = await this.getBookList({search: value, limit: 15, shareStatus: 'SHARED'})
    }, 500),
    async getBookList({userId, shareStatus, limit}) {
	  const data = await this.$api.post('/api2/books/shareBooks/getBookList', {
	    search: this.key.trim(),
	    shareStatus,
	    limit
	  })
	  return data
    },
    inputKey: function(e) {
      this.key = util.trim(e.detail.value)
      if (!this.key) {
        this.searchList = []
        return
      }
      // 根据关键词查找
      let arr = []
      // 实际开发中，根据搜索返回结果集，此处只是做展示提示搜索哪些文字
      this.searchResult.forEach((item) => {
        arr.push({
          key: item,
          showKey: util.replaceAll(item, this.key, `<label style="color:#E41F19">${this.key}</label>`)
        })
      })
      this.searchList = arr
    }
  }
}
</script>

<style lang='scss'>

	page {
		color: #333;
		background: #fff;
	}

	.container {
	  background: #fff;
	  padding: 0 30rpx 30rpx 30rpx;
	  box-sizing: border-box;
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

	.tui-searchbox {
		padding: 30rpx 0;
		box-sizing: border-box;
		display: flex;
		align-items: center;
	}

	.tui-search-input {
		width: 100%;
		height: 66rpx;
		border-radius: 35rpx;
		padding: 0 30rpx;
		box-sizing: border-box;
		background: #f2f2f2;
		display: flex;
		align-items: center;
		flex-wrap: nowrap;
	}

	.tui-input {
		flex: 1;
		color: #333;
		padding: 0 16rpx;
		font-size: 28rpx;
	}

	.tui-input-plholder {
		font-size: 28rpx;
		color: #b2b2b2;
	}

	.tui-cancle {
		color: #888;
		font-size: 28rpx;
		padding-left: 30rpx;
		flex-shrink: 0;
	}

	.tui-history-header {
		display: flex;
		align-items: center;
		justify-content: space-between;
		padding: 30rpx 0;
	}
	.tui-history-content{
		display: flex;
		align-items: center;
		flex-wrap: wrap;
	}

	.tui-icon-delete {
		padding: 10rpx;
	}

	.tui-search-title {
		font-size: 28rpx;
		font-weight: bold;
	}

	.tui-hot-header {
		padding: 30rpx 0;
	}

	.tui-header {
		padding: 26rpx 0;
		box-sizing: border-box;
		display: flex;
		align-items: center;
		justify-content: space-between;
	}

	.tui-header-left {
		font-size: 30rpx;
		color: #222;
		border-left: 4rpx solid #EB0909;
		padding-left: 10rpx;
		word-break: break-all;
	}

	.tui-noboredr {
		border-left: 0 !important;
	}

	.tui-result-box {
		font-size: 28rpx;
	}

	.tui-result-item {
		line-height: 28rpx;
		padding: 28rpx 0;
	}

</style>
