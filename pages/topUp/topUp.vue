<template>
	<view>
		<view class="content">
			<view class="title">快速充值</view>
			<view class="recharge">
				<view class="box" @click="change(1)">
					<view :class="currentItem == 1 ? 'price active' : 'price'">
						￥
						<text>200</text>
					</view>
					<view :class="currentItem == 1 ? 'selling active' : 'selling'">售价200元</view>
				</view>
				<view class="box" @click="change(2)">
					<view :class="currentItem == 2 ? 'price active' : 'price'">
						￥
						<text>500</text>
					</view>
					<view :class="currentItem == 2 ? 'selling active' : 'selling'">售价500元</view>
				</view>
				<view class="box" @click="change(3)">
					<view :class="currentItem == 3 ? 'price active' : 'price'">
						￥
						<text>1000</text>
					</view>
					<view :class="currentItem == 3 ? 'selling active' : 'selling'">售价1000元</view>
				</view>
			</view>
			<view class="title">自定义金额</view>
			<uni-easyinput focus v-model="value" @change="setMoneys" class="uni-input" type="number" placeholder="请输入金额" />
			<view class="title">充值说明</view>
			<view class="text">
				<view>1、充值业务一般是准实时到账</view>
				<view>2、如遇异常情况，可以及时联系客服处理</view>
				<view>3、不排除存在不可控原因影响资金到账时效</view>
				<view>4、请在“转入转出记录”中点击查看触发后台主动</view>
			</view>
		</view>
		<view class="submit" @click="submit"><view>提交</view></view>
		<uni-popup ref="popup" type="dialog">
			<uni-popup-dialog
				type="success"
				cancelText="关闭"
				title="通知"
				content="充值成功!"
				@confirm="close"
			></uni-popup-dialog>
		</uni-popup>
	</view>
</template>
<script>
export default {
	components: {},
	data() {
		return {
			currentItem: 1,
			money: 200,
			value: null,
			userId: null,
			balance: null
		};
	},
	onLoad(option) {
		this.userId = option.id;
		this.money = 200;
	},
	methods: {
		back() {
			uni.navigateBack({});
		},
		submit() {
			console.log('充值的金额：' + this.money);
			if (this.money == null) {
				this.money = 200;
			}
			uni.request({
				url: 'http://localhost:8088/user/balanceUpd?id=' + this.userId + '&balance=' + parseFloat(this.money),
				method: 'get',
				success: res => {
					this.balance = res.data;
					uni.setStorageSync('balance', res.data);
					this.$refs.popup.open();
				}
			});
		},
		close() {
			// TODO 做一些其他的事情，before-close 为true的情况下，手动执行 close 才会关闭对话框
			// ...
			this.$refs.popup.close();
		},
		change(e) {
			this.currentItem = e;
			this.value = null;
			if (e == 1) this.money = 200;
			if (e == 2) this.money = 500;
			if (e == 3) this.money = 1000;
		},
		setMoneys() {
			this.money = this.value;
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
.title {
	font-size: 32rpx;
	font-weight: 600;
	color: #222222;
	margin-bottom: 32rpx;
}
.recharge {
	display: flex;
	justify-content: space-between;
	align-items: center;
	margin-bottom: 44rpx;
	.box {
		width: 208rpx;
		height: 146rpx;
		background: #ffffff;
		border-radius: 8rpx;
		text-align: center;

		.price {
			font-size: 24rpx;
			font-weight: 600;
			color: #333333;
			margin: 24rpx 0 8rpx 0;
			text {
				font-size: 28rpx;
			}
		}
		.selling {
			font-size: 28rpx;
			font-weight: 400;
			color: #999999;
		}
		.active {
			color: #e8000a;
		}
	}
}
/deep/ .uni-easyinput__content,
.uni-input {
	width: 686rpx;
	height: 104rpx;
	background: #ffffff;
	border-radius: 8rpx;
	padding: 32rpx;
	box-sizing: border-box;
	margin-bottom: 32rpx;
}
.submit {
	width: 100%;
	height: 148rpx;
	background: #ffffff;
	position: fixed;
	left: 0;
	right: 0;
	bottom: 0;

	view {
		width: 686rpx;
		height: 84rpx;
		line-height: 84rpx;
		border-radius: 8rpx;
		background: #e8000a;
		color: #ffffff;
		text-align: center;
		margin: 32rpx auto 0;
	}
}
.text {
	background: #ffffff;
	border-radius: 8rpx;
	padding: 32rpx;
	view {
		font-size: 28rpx;
		font-weight: 600;
		color: #222222;
		margin-bottom: 16rpx;
	}
	view:last-child {
		margin-bottom: 0;
	}
}
</style>
