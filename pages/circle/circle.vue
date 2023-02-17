<template>
	<div>
		<div class="page">
			<div class="circle">
				<van-circle size="200" stroke-width="20" :rate="index" :speed="100/(index/1000)"
					v-model:current-rate="currentRate">
					<van-count-down :use-slot="true" format="HH:mm:ss:SSS" :auto-start="auto" @finished="finished" class="control-count-down"
						style="font-size: 30px;" :time="index" @change="onChange">
						<slot>
							<text class="item">{{ timeData.hours }}:</text>
							<text class="item">{{ timeData.minutes }}:</text>
							<text class="item">{{ timeData.seconds }}</text>
						</slot>
					</van-count-down>
				</van-circle>
				<view class="weui-btn-area">
					<van-button round plain icon="contact" type="info" open-type="contact">
						联系客服
					</van-button>
				</view>
			</div>
			<div style="display: flex; justify-content:space-around; color:#fff">
				<div style="text-align: center;">
					<div style="font-size: 14px;">预付金额</div>
					<div style="font-size: 26px;">￥{{spend}}</div>
				</div>
				<div style="text-align: center;">
					<div style="font-size: 14px;">预充时长</div>
					<div style="font-size: 26px;">{{prtime}}</div>
				</div>
			</div>
		</div>
		<view class="endCharge">
			<van-button size="large" style="width: 200px;" @click="pause" round type="info">
				结束充电
			</van-button>
		</view>
	</div>
</template>

<script>
	// pages/order/order.js
	export default {
		components: {},
		data() {
			return {
				rate: '',
				speed: '',
				time: '',
				prtime:'',
				currentRate: 0,
				index: '',
				spend: '',
				order: {},
				timer:'',
				timeData:{},
				auto:false,
				range: ['30分钟', '1小时', '1.5小时', '2小时', '2.5小时', '3小时', '3.5小时', '4小时', '4.5小时', '5小时', '5.5小时', '6小时', '4.5小时', '5小时', '5.5小时', '6小时'],
			};
		},
		/**
		 * 生命周期函数--监听页面加载
		 */
		onLoad(options) {
			this.spend = options.spend
			this.time = options.time
			console.log(this.time)
			this.prtime = this.range[this.time]
			this.id = options.id
			console.log(options.index)
			this.index = (options.index<10)?(options.index + 1) * 0.5 * 60 * 60 * 1000:options.index
			console.log(this.index)
			if(options.index<10){
				this.start()
			}else{
				this.auto = true
			}
		},

		methods: {
			// 暂停录音
			suspendFn() {
				clearInterval(this.timer)
			},
			onChange(e){
				this.timeData = e.detail
				this.currentRate =Math.round(100-(((this.timeData.hours*60+this.timeData.minutes)*60+this.timeData.seconds)/((this.time + 1) * 0.5 * 60 * 60)*100));
				console.log(this.currentRate)
			},
			start() {
				uni.showModal({
					content: "请确定电源已接入充电设备？",
					showCancel: false,
					success: (res) => {
						if (res.confirm) {
							const countDown = this.selectComponent('.control-count-down');
							countDown.start();
						} else if (res.cancel) {
							console.log('用户点击取消');
						}
					}
				})
			},

			pause() {
				const countDown = this.selectComponent('.control-count-down');
				countDown.pause();
				this.order.state = 1
				this.order.spend = this.spend * this.currentRate / 100
				this.order.id = this.id
				uni.request({
					url: 'http://localhost:8088/order/orderUpd',
					method: 'post',
					data: this.order,
					success: (res) => {
						uni.navigateTo({
						    url: '/pages/map/map'
						});
					}
				});
			},
			finished() {
				uni.showToast({
					title: "充电结束！"
				})
			}
		}
	};
</script>
<style>
	@import './circle.css';
</style>
