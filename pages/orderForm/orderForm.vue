<template>
    <view class="page">
        <div id="form">
            <van-cell-group>
                <van-field :value="marker.name" name="name" label="名称"></van-field>
                <van-field :value="marker.address" label="地址"></van-field>
                <van-field :value="marker.price" label="价格"></van-field>
                <van-field v-if="marker.type == '1'" label="类型">
                    <template #input>
                        <van-radio-group @change="formRadioChange" :value="order.type" direction="horizontal">
                            <van-radio name="30">30kw</van-radio>
                            <van-radio name="60">60kw</van-radio>
                        </van-radio-group>
                    </template>
                </van-field>
                <van-field v-if="marker.type == '0'" label="类型" value="慢充（10kw）"></van-field>
                <van-field clickable label="时间" :value="time" placeholder="选择时间" @click-input="showPicker" />
                <van-popup :show="show" round position="bottom" :style="{ height: '30%' }">
                    <van-picker show-toolbar :columns="range" @cancel="show = false" @confirm="timeChange" />
                </van-popup>
                <van-field label="总计" :value="spend"></van-field>
            </van-cell-group>
            <view class="weui-btn-area"><button class="weui-btn" type="primary" @tap="submitForm">提交</button></view>
        </div>
    </view>
</template>

<script>
const { formatTime } = require('../../utils/util');

// pages/orderForm/orderForm.js
export default {
    components: {},
    data() {
        return {
            marker: {
                name: '',
                address: '',
                price: '',
                type: ''
            },

            userId: '',
            range: ['30分钟', '1小时', '1.5小时', '2小时', '2.5小时', '3小时', '3.5小时', '4小时', '4.5小时', '5小时', '5.5小时', '6小时', '4.5小时', '5小时', '5.5小时', '6小时'],
            time: '请选择时间',
            index: '',
            power: '',
            order: {},
            spend: '0',
            rules: [],
            type: '',
            show: false
        };
    },
    /**
     * 生命周期函数--监听页面加载
     */
    onLoad(options) {
        this.setData({
            marker: JSON.parse(options.data)
        });
        this.userId = uni.getStorageSync('userId');
    },
    /**
     * 生命周期函数--监听页面初次渲染完成
     */
    onReady() {},
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
        showPicker() {
            this.show = true;
        },
        timeChange(e) {
            console.log(e);
            this.time = e.detail.value;
            this.index = e.detail.index;
            this.show = false;
            if (this.power != '') {
                let power1;
                if (this.marker.type == '1') {
                    power1 = parseInt(this.power);
                } else {
                    power1 = 10;
                }
                let spend = (parseInt(this.index) + 1) * 0.5 * power1 * parseFloat(this.marker.price);
                this.spend = spend;
            }
        },

        formRadioChange(e) {
            console.log(e);
            this.power = e.detail;
            if (this.index != '') {
                let power1;
                if (this.marker.type == '1') {
                    power1 = parseInt(e.detail);
                } else {
                    power1 = 10;
                }
                this.spend = (parseInt(this.index) + 1) * 0.5 * power1 * parseFloat(this.marker.price);
            }
        },

        submitForm() {
            let startTime = formatTime(new Date());
            let time = new Date().getTime() + (parseInt(this.index) + 1) * 0.5 * 60 * 60 * 1000;
            let endTime = formatTime(new Date(time));
            console.log(endTime);
            this.order.endTime = endTime;
            this.order.startTime = startTime;
            this.order.chargeId = this.marker.id;
            this.order.userId = this.userId;
            this.order.spend = this.spend;
            this.order.state = 0;
            uni.request({
                url: 'http://localhost:8088/order/orderAdd',
                method: 'post',
                data: this.order,
                success: res => {
                    uni.redirectTo({
                        url: '/pages/circle/circle?spend=' + this.spend + '&time=' + this.index + '&index=' + this.index + '&id=' + res.data
                    });
                }
            });
        }
    }
};
</script>
<style lang="scss" scoped>
@import './orderForm.css';
.page {
    background: #f9f9f9;
    height: 100vh;
}
#form {
    background: #ffffff;
    border-radius: 16rpx;
    padding: 10px 3px;
    margin-top: 16rpx;
}
.weui-btn-area {
    display: flex;
    justify-content: center;
    position: absolute;
    bottom: 56rpx;
    width: 750rpx;
    height: 168rpx;
    background: #ffffff;
    .weui-btn {
        width: 686rpx;
        height: 88rpx;
        background: #1890ff;
        border-radius: 44rpx;
    }
}
</style>
