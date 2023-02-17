<template>
	<div class="container">
		<div class="header">
			<div style="display: flex; align-items:center;padding: 0 0 0 10px;">
				<van-icon name="user-circle-o" @click="toUser" style="font-size: 24px;"></van-icon>
				<van-search class="search" placeholder="输入电站名" v-model="value" @search="onSearch" @cancel="onCancel"></van-search>
			</div>
			<van-dropdown-menu>
				<van-dropdown-item v-model="value1" :options="option1" @change="stateSelect" />
				<van-dropdown-item v-model="value2" :options="option2" @change="typeSelect" />
			</van-dropdown-menu>
		</div>
		<div
			@tap="chargeStart"
			style="width: 65px; display: flex; justify-content: center; align-items: center; flex-wrap: wrap; position: absolute; top: 150px; right: 30px; z-index: 1"
		>
			<image style="width: 50px; height: 50px" src="/static/pages/image/sxb-充电宝.png" />
			<text>开始充电</text>
		</div>
		<van-notice-bar v-if="noticeFlag" text="存在待支付订单!!!" />
		<map
			id="1"
			style="width: 100%; height: 100vh"
			:subkey="subkey"
			scale="18"
			:markers="markers"
			:longitude="longitude"
			:latitude="latitude"
			show-location
			@markertap="markertap"
		></map>
		<van-action-sheet :show="show" @click-overlay="bindClose" close-on-click-overlay="true">
			<div class="content">
				<van-card style="font-size: 16px;" :title="marker.name" :desc="marker.address" :price="marker.price + '元/度'">
					<template #tags>
						<van-tag v-if="marker.type == '1'" type="primary">快{{ marker.num }}/{{ marker.sum }}</van-tag>
						<van-tag v-if="marker.type == '0'" type="warning">慢{{ marker.num }}/{{ marker.sum }}</van-tag>
					</template>
					<template #footer>
						<div style="display: flex; align-items:baseline; justify-content:space-between;">
							<div style="display: flex; align-items:baseline; " @click="collectionFun">
								<image style="width: 20px; height: 20px; " src="../../static/pages/image/收藏.png"></image>
								收藏
							</div>
							<van-button round style="display: flex; align-items:baseline; " @click="navigation">
								<image style="width: 20px; height: 20px; " src="../../static/pages/image/导航.png"></image>
								导航
							</van-button>
						</div>
					</template>
					<template #footer>
						<van-button icon="star-o" round size="mini" type="default" @click="collectionFun">收藏</van-button>
						<van-button icon="guide-o" round size="mini" type="primary" @click="navigation">导航</van-button>
					</template>
				</van-card>
			</div>
		</van-action-sheet>
	</div>
</template>

<script>
// pages/map/map.js
export default {
	components: {},
	data() {
		return {
			subkey: 'RYEBZ-LHIKV-JKQPP-UTLDY-DMEJH-KQFQK',
			longitude: '',
			latitude: '',
			markers: [],
			circles: [],
			show: false,
			buttons: [],
			markerId: '',

			marker: {
				name: '',
				address: '',
				price: '',
				sum: '',
				num: '',
				type: ''
			},
			list: [],
			userId: '',

			collection: {
				userId: '',
				chargeId: ''
			},

			index: '0',
			value: '',
			option1: [{ text: '充电站', value: '0' }, { text: '个人桩', value: '1' }, { text: '全部', value: '2' }],
			option2: [{ text: '慢充', value: '0' }, { text: '快充', value: '1' }, { text: '筛选', value: '2' }],
			value1: '2',
			value2: '2',
			noticeFlag: false
		};
	},
	/**
	 * 生命周期函数--监听页面加载
	 */
	onLoad(options) {
		this.userId = uni.getStorageSync('userId');
		uni.getLocation({
			isHighAccuracy: true,
			type: 'wgs84',
			success: res => {
				(this.latitude = res.latitude), (this.longitude = res.longitude);
			}
		});
		uni.request({
			url: 'http://localhost:8088/charge/getCharges',
			method: 'GET',
			success: res => {
				res.data.forEach(e => {
					e.iconPath = '/static/pages/image/充电器.png';
					e.width = 30;
					e.height = 30;
				});
				this.markers = res.data;
				this.list = res.data;
			}
		});
		uni.request({
			url: 'http://localhost:8088/order/getOrdersByUserId?userId=' + this.userId,
			method: 'get',
			success: res => {
				console.log(res);
				res.data.forEach(item => {
					if (item.state === 1) {
						this.noticeFlag = true;
						return;
					}
				});
			}
		});
	},
	/**
	 * 生命周期函数--监听页面初次渲染完成
	 */
	onReady() {
		const map = uni.createMapContext('map');
		map.moveToLocation();
	},
	/**
	 * 生命周期函数--监听页面显示
	 */
	onShow() {},
	/**
	 * 生命周期函数--监听页面隐藏
	 */
	onHide() {},
	/**
	 * 生命周期函数--监听页面卸载
	 */
	onUnload() {},
	/**
	 * 页面相关事件处理函数--监听用户下拉动作
	 */
	onPullDownRefresh() {},
	/**
	 * 页面上拉触底事件的处理函数
	 */
	onReachBottom() {},
	/**
	 * 用户点击右上角分享
	 */
	onShareAppMessage() {},
	methods: {
		toUser() {
			uni.navigateTo({
				url: '/pages/user/user'
			});
		},
		chargeStart() {
			if (this.noticeFlag == true) {
				uni.showToast({
					title: '存在待支付订单，请先完成支付',
					icon: 'error'
				});
				return;
			}
			uni.showModal({
				editable: true,
				placeholderText: '请输入终端编号',
				success: res => {
					this.markers.forEach(element => {
						if (element.code === res.content) {
							this.setData({
								marker: element
							});
							return;
						}
					});
					if (this.userId == '') {
						uni.showToast({
							title: '请先登录',
							icon: 'error'
						});
					} else {
						let data = JSON.stringify(this.marker);
						uni.navigateTo({
							url: '/pages/orderForm/orderForm?data=' + data + '&userId=' + this.userId
						});
					}
				}
			});
		},
		markertap(e) {
			(this.show = true), (this.markerId = e.detail.markerId);
			this.markers.forEach(element => {
				if (element.id === e.detail.markerId) {
					this.marker = element;
					return;
				}
			});
		},

		navigation() {
			console.log(this.marker);
			var lat = parseFloat(this.marker.latitude);
			var lon = parseFloat(this.marker.longitude);
			let plugin = requirePlugin('routePlan');
			let key = 'RYEBZ-LHIKV-JKQPP-UTLDY-DMEJH-KQFQK';
			let referer = 'wx0e69a669d6692dd1';
			let endPoint = JSON.stringify({
				//终点
				name: this.marker.address,
				latitude: lat,
				longitude: lon
			});

			console.log(endPoint);
			// uni.navigateTo({
			// 	url: 'plugin://routePlan/index?key=' + key + '&referer=' + referer + '&endPoint=' + endPoint + '&navigation=1',
			// 	success(res){
			// 		console.log(res)
			// 	},
			// 	fail(res) {
			// 		console.log("error"+res)
			// 	}

			// });
			uni.getLocation({
				type: 'gcj02', // 默认为 wgs84 返回 gps 坐标，gcj02 返回可用于 wx.openLocation 的坐标
				success: (res)=> {
					// let that = this
					uni.openLocation({
						latitude: Number(lat), // 纬度，范围为-90~90，负数表示南纬
						longitude: Number(lon), // 经度，范围为-180~180，负数表示西经
						scale: 28, // 缩放比例
						name: this.marker.address,
						address: this.marker.address
					});
				}
			});
		},

		bindClose: function() {
			this.show = false;
		},

		collectionFun() {
			if (this.userId == 0 || this.userId == '') {
				uni.showToast({
					title: '请先登录',
					icon: 'error'
				});
			} else {
				this.collection.userId = this.userId;
				this.collection.chargeId = this.markerId;
				uni.request({
					url: 'http://localhost:8088/collection/CollectionAdd',
					method: 'post',
					data: this.collection,
					success: res => {
						uni.showToast({
							title: '已收藏'
						});
						this.show = false;
					}
				});
			}
		},

		onSearch: function(value) {
			this.markers = this.list;
			this.markers = this.markers.filter(marker => {
				return marker.name.indexOf(value.detail) != -1;
			});
			console.log(this.markers);
		},
		onCancel: function() {
			this.value = '';
		},
		stateSelect(value) {
			this.markers = this.list;
			if (value.detail !== '2') {
				this.markers = this.markers.filter(marker => {
					return marker.state === value.detail;
				});
			}
			console.log(this.markers);
		},
		typeSelect(value) {
			this.markers = this.list;
			if (value.detail !== '2') {
				this.markers = this.markers.filter(marker => {
					return marker.type === value.detail;
				});
			}
			console.log(this.markers);
		}
	}
};
</script>
<style>
@import './map.css';
</style>
