<template>
	<view>
		<view class="content">
			<view class="money">
				<view class="left">
					<view class="me">我的余额</view>
					<view class="balance">
						￥
						<text>{{balance}}</text>
					</view>
					<view class="detail" @click="clickDetail">查看详情</view>
				</view>
				<view class="right"><img src="@/static/pages/image/money.png" alt="" /></view>
			</view>
			<view class="pay" @click="toPay">充值</view>
		</view>
	</view>
</template>

<script>
export default {
	components: {
		// customNavagition
	},
	data() {
		return {
			balance: null,
			userId: null
		};
	},
	onLoad(option) {
		this.userId = option.id;
		uni.request({
		    url: 'http://localhost:8088/user/getBalance?id=' + option.id,
		    method: 'get',
		    success: (res) => {
		        this.setData({
		            balance: res.data
		        });
				
		    }
		});
	},
	methods: {
		back() {
			uni.navigateBack({
				delta: 1
			});
		},
		toPay() {
			uni.navigateTo({
				url: '/pages/topUp/topUp?id='+this.userId
			});
		},
		clickDetail() {
			// uni.navigateTo({
			// 	url: '/userInfo/recordBalance/recordBalance'
			// });
		}
	}
};
</script>

<style lang="scss" scoped>
.navagition-icon {
	position: absolute;
	bottom: 18rpx;
	left: 32rpx;
}
.content {
	padding: 32rpx;
}
.money {
	width: 686rpx;
	height: 336rpx;
	background: #f64d43;
	border-radius: 16rpx;
	padding: 48rpx;
	box-sizing: border-box;
	display: flex;
	justify-content: space-between;
	color: #ffffff;
	.me {
		font-size: 24rpx;
		font-weight: 400;
		margin-bottom: 16rpx;
	}
	.balance {
		font-size: 24rpx;
		font-weight: 400;
		text {
			font-size: 28rpx;
			font-weight: 600;
		}
	}
	.detail {
		width: 112rpx;
		height: 44rpx;
		border-radius: 8rpx;
		border: 2rpx solid #ffffff;
		font-size: 20rpx;
		font-weight: 400;
		text-align: center;
		line-height: 44rpx;
		margin-top: 88rpx;
	}
	.right {
		img {
			width: 208rpx;
			height: 208rpx;
		}
	}
}
.pay,
.withDraw {
	width: 686rpx;
	height: 84rpx;
	background: #ffffff;
	border-radius: 8rpx;
	text-align: center;
	line-height: 84rpx;
	color: #666666;
	font-size: 28rpx;
	font-weight: 400;
}
.pay {
	margin: 48rpx 0 32rpx 0;
	color: #ff0000;
}
</style>
