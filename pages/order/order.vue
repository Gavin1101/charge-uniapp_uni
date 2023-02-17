<template>
    <view class="page">
		<van-tabs :active="active" @change="onChange">
		  <van-tab title="全部"></van-tab>
		  <van-tab title="进行中"></van-tab>
		  <van-tab title="待支付"></van-tab>
		  <van-tab title="已完成"></van-tab>
		</van-tabs>
		<van-card style="margin: 5;font-size: 16px;" v-for="(item, index) in orders"
		            :key="index" :title="item.name" :desc="item.address" :price="item.spend">
			<template #tags>
				<van-tag v-if="item.state=='2'" type="info">已完成</van-tag>
				<van-tag v-if="item.state=='1'" type="warning">待支付</van-tag>
				<van-tag v-if="item.state=='0'" type="primary">进行中</van-tag>
			</template>
			<template #footer>
				<van-button v-if="item.state===1" round size="mini" type="primary" @click="pay(item)">支付</van-button>
				<van-button v-if="item.state===1" round size="mini" type="info" @click="myCash">充值</van-button>
				<van-button v-if="item.state===2" round size="mini" type="danger" @click="orderDel(item)">删除</van-button>
				<van-count-down v-if="item.state===0" @finished="finished" class="control-count-down"
					 :time="item.time">
				</van-count-down>
				<van-button v-if="item.state===0" round size="mini" type="info" @click="toCircle(item)">查看</van-button>
			</template>
		</van-card>
    </view>
</template>

<script>
// pages/order/order.js
export default {
    components: {},
    data() {
        return {
            userId: '',
            orders: [],
			list:[],
			range: ['30分钟', '1小时', '1.5小时', '2小时', '2.5小时', '3小时', '3.5小时', '4小时', '4.5小时', '5小时', '5.5小时', '6小时', '4.5小时', '5小时', '5.5小时', '6小时'],
            order: {
                name: '',
                price: '',
                startTime: '',
                endTime: '',
                spend: ''
            },
            show: false,
			active:''
        };
    },
    /**
     * 生命周期函数--监听页面加载
     */
    onLoad(options) {
        this.userId = uni.getStorageSync("userId")
    },
    /**
     * 生命周期函数--监听页面初次渲染完成
     */
    onReady() {
        uni.request({
            url: 'http://localhost:8088/order/getOrdersByUserId?userId=' + this.userId,
            method: 'get',
            success: (res) => {
				res.data.forEach((item)=>{
					if(item.state === 0){
						let time = new Date(item.endTime).getTime()-new Date().getTime()
						if(time<0){
							item.state = 1
							uni.request({
								url: 'http://localhost:8088/order/orderUpd',
								method: 'post',
								data: item,
								success: (res) => {}
							});
						}else{
							item.time = time
						}
					}
				})
                this.orders = res.data
				this.list = res.data
            }
        });
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
		toCircle(item){
			console.log(item.state)
			if(item.state===0){
				let time = new Date(item.endTime).getTime()-new Date(item.startTime).getTime()
				let index = time/1000/60/30-1
				uni.redirectTo({
					url: '/pages/circle/circle?spend='+item.spend+'&time='+index+'&index='+item.time+'&id='+item.id
				});
			}
		},
		onChange(e){
			if(e.detail.index===1){
				this.orders = this.list.filter(item=>{
					return item.state === 0
				})
				console.log(this.orders)
			}else if(e.detail.index===2){
				this.orders = this.list.filter(item=>{
					return item.state === 1
				})
				console.log(this.orders)
			}else if(e.detail.index===3){
				this.orders = this.list.filter(item=>{
					return item.state === 2
				})
				console.log(this.orders)
			}else{
				this.orders = this.list
			}	
		},
        orderDel(item) {
            uni.request({
                url: 'http://localhost:8088/order/orderDel?id=' + item.id + '&userId=' + item.userId,
                method: 'get',
                success: (res) => {
                    this.orders = res.data
                    this.list = res.data
                }
            });
        },
		pay(item){
			var balance = uni.getStorageSync("balance")
			if(balance<item.spend){
				uni.showToast({
					title: '余额不足！',
					icon:"error"
				})
				return
			}
			uni.request({
			    url: 'http://localhost:8088/user/balanceUpd?id=' + item.userId + '&balance=' + parseFloat(item.spend) * -1,
			    method: 'get',
			    success: (res) => {
			        uni.setStorageSync('balance', res.data);
			    }
			});
			item.state = 2
			uni.request({
				url: 'http://localhost:8088/order/orderUpd',
				method: 'post',
				data: item,
				success: (res) => {
					// uni.navigateTo({
					//     url: '/pages/map/map'
					// });
				}
			});
		},
		myCash() {
		    uni.showModal({
		        editable: true,
		        placeholderText: '请输入充值额度',
		        success: (res) => {
		            uni.request({
		                url: 'http://localhost:8088/user/balanceUpd?id=' + this.userId + '&balance=' + parseFloat(res.content),
		                method: 'get',
		                success: (res) => {
							this.balance = res.data;
		                    uni.setStorageSync('balance', res.data);
		                }
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
@import './order.css';
</style>
