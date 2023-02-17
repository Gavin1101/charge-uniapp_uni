<template>
    <view class="page">
        <button @tap="chargeAdd">新增</button>
		<van-card style="margin: 5; font-size: 16px;" v-for="(item, index) in charges"
                    :key="item.id" :title="item.name" :desc="item.address" :price="item.price+'元/度'">
			<template #tags>
				<van-tag v-if="item.type=='1'" type="primary">快{{item.num}}/{{item.sum}}</van-tag>
				<van-tag v-if="item.type=='0'" type="warning">慢{{item.num}}/{{item.sum}}</van-tag>
			</template>
			<template #footer>
				<van-button round size="mini" type="primary" @click="chargeUpd(item)">编辑</van-button>
				<van-button round size="mini" type="danger" @click="chargeDel(item.id)">删除</van-button>
			</template>
		</van-card>
    </view>
</template>

<script>
// pages/charge/charge.js
export default {
    components: {},
    data() {
        return {
            userId: '',
            userName: '',
            charges: [],
        };
    },
    /**
     * 生命周期函数--监听页面加载
     */
    onLoad(options) {
		this.slideButtons=[
                {
                    text: '删除',
                    type: 'warn'
                }
            ];
		this.userId = uni.getStorageSync("userId")
		this.userName = uni.getStorageSync("username")
    },
    /**
     * 生命周期函数--监听页面初次渲染完成
     */
    onReady() {
        uni.request({
            url: 'http://localhost:8088/charge/getChargesByUserId?userId=' + this.userId,
            method: 'get',
            success: (res) => {
                console.log(res.data);
				this.charges=res.data;
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
    onUnload() {
        uni.navigateTo({
            url: '/pages/user/user'
        });
    },
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
        chargeAdd() {
            uni.navigateTo({
                url: '/pages/chargeForm/chargeForm'
            });
        },

        chargeUpd(item) {
            let data = JSON.stringify(item);
            uni.navigateTo({
                url: '/pages/chargeForm/chargeForm?data=' + data
            });
        },

        chargeDel(id) {
            uni.request({
                url: 'http://localhost:8088/charge/chargeDel?id=' + id,
                method: 'get',
                success: (res) => {
                    this.setData({
                        show: false
                    });
                    uni.request({
                        url: 'http://localhost:8088/charge/getChargesByUserId?userId=' + this.userId,
                        method: 'get',
                        success: (res) => {
                            console.log(res.data);
                            this.setData({
                                charges: res.data
                            });
                        }
                    });
                }
            });
        }
    }
};
</script>
<style>
@import './myCharge.css';
</style>
