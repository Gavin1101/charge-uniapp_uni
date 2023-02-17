<template>
    <view class="page">
        <van-card style="margin: 5;font-size: 16px;" v-for="(item, index) in collections"
                    :key="index" :title="item.name" :desc="item.address" :price="item.price+'元/度'">
        	<template #tags>
        		<van-tag v-if="item.type=='1'" type="primary">快{{item.num}}/{{item.sum}}</van-tag>
        		<van-tag v-if="item.type=='0'" type="warning">慢{{item.num}}/{{item.sum}}</van-tag>
        	</template>
        	<template #footer>
				<van-button round size="mini" type="warning" @click="collectionDel(item.collectionId)">取消收藏</van-button>
				<van-button icon="guide-o" round size="mini" type="primary" @click="navigation(item)">导航</van-button>
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
            slideButtons: [],
            collections: [],
            show: false,

            marker: {
                name: '',
                price: '',
                sum: '',
                num: '',
                type: ''
            },

            userId: ''
        };
    },
    /**
     * 生命周期函数--监听页面加载
     */
    onLoad(options) {
        this.userId = uni.getStorageSync("userId")
        this.setData({
            slideButtons: [
                {
                    text: '删除',
                    type: 'warn'
                }
            ],
        });
    },
    /**
     * 生命周期函数--监听页面初次渲染完成
     */
    onReady() {
        uni.request({
            url: 'http://localhost:8088/collection/getChargesByUserId?userId=' + this.userId,
            method: 'get',
            success: (res) => {
                console.log(res.data);
				this.collections=res.data
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
        collectionDel(id) {
            uni.request({
                url: 'http://localhost:8088/collection/CollectionDel?id=' + id + '&userId=' + this.userId,
                method: 'get',
                success: (res) => {
                    console.log(res.data);
                    this.setData({
                        collections: res.data,
                    });
                }
            });
        },
		navigation(item) {
		    var lat = parseFloat(item.latitude);
		    var lon = parseFloat(item.longitude);
		    let plugin = requirePlugin('routePlan');
		    let key = 'C5XBZ-CXJCG-MRLQQ-IRD4H-2EIWT-M3FZ2';
		    let referer = 'wx00891957d641a086';
		    let endPoint = JSON.stringify({
				name: item.address,
		        latitude: lat,
		        longitude: lon
		    });
		    uni.navigateTo({
		        url: 'plugin://routePlan/index?key=' + key + '&referer=' + referer + '&endPoint=' + endPoint
		    });
		},
    }
};
</script>
<style>
@import './charge.css';
</style>
