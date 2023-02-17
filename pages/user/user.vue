<template>
    <view>
        <view class="header">
            <view class="mine" v-if="!isLogin">
                <button class="btn" @tap="login">请点击授权登录</button>
            </view>
            <view class="mine" v-else-if="isLogin">
                <open-data class="avatar" type="userAvatarUrl"></open-data>
            </view>
        </view>
        <view class="card">
            <view class="title">我的钱包</view>
            <view class="funList">
                <view class="item" @tap="myBalance">
                    <image class="img" src="/static/pages/image/余额.png"></image>
                    <text>余额</text>
                </view>
                <view class="item" @tap="myCash">
                    <image class="img" src="/static/pages/image/充值.png"></image>
                    <text>充值</text>
                </view>
            </view>
        </view>
        <view class="card">
            <view class="title">我的充电</view>
            <view class="funList">
                <view class="item" @tap="myCollection">
                    <image class="img" src="/static/pages/image/收藏.png"></image>
                    <text>收藏</text>
                </view>
                <view class="item" @tap="myOrder">
                    <image class="img" src="/static/pages/image/订单.png"></image>
                    <text>订单</text>
                </view>
                <view class="item" @tap="myCharge">
                    <image class="img" src="/static/pages/image/充电站管理.png"></image>
                    <text>充电桩</text>
                </view>
            </view>
        </view>
    </view>
</template>

<script>
// pages/user/user.js
export default {
    data() {
        return {
            userInfo: {},
            userInfoStr: '',
            isLogin: false,
            userId: '',
            collectons: [],
            balance: ''
        };
    }
    /**
     * 生命周期函数--监听页面加载
     */,
    onLoad(options) {
        this.setData({
            balance: uni.getStorageSync('balance')
        });
		this.userId = uni.getStorageSync("userId")
		this.isLogin = uni.getStorageSync("isLogin")
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
        myCollection() {
            if (this.userId == '') {
                uni.showToast({
                    title: '请先登录',
                    icon: 'error'
                });
            } else {
                uni.navigateTo({
                    url: '/pages/charge/charge'
                });
            }
        },

        myCharge() {
            if (this.userId == '') {
                uni.showToast({
                    title: '请先登录',
                    icon: 'error'
                });
            } else {
                uni.navigateTo({
                    url: '/pages/myCharge/myCharge'
                });
            }
        },

        myCash() {
           if (this.userId != '') {
           	uni.navigateTo({
           		url: '/pages/topUp/topUp?id='+this.userId
           	})
           }
        },

        myBalance() {
            if (this.userId != '') {
				uni.navigateTo({
					url: '/pages/balance/balance?id='+this.userId
				})
            }
        },

        myOrder() {
            if (this.userId == '') {
                uni.showToast({
                    title: '请先登录',
                    icon: 'error'
                });
            } else {
                uni.navigateTo({
                    url: '/pages/order/order'
                });
            }
        },

        login: function () {
            uni.getUserProfile({
                desc: '用于微信账号与平台账号绑定',
                success: (res) => { 
					this.userInfo=res.userInfo;
					this.userInfoStr=JSON.stringify(res);
					this.isLogin=true;
                    uni.request({
                        url: 'http://localhost:8088/user/login?name=' + res.userInfo.nickName,
                        method: 'get',
                        success: (res) => {
                            uni.setStorageSync('userId', res.data);
							uni.setStorageSync('username', this.userInfo.nickName);
							uni.setStorageSync('isLogin', this.isLogin);
							this.userId=res.data;
                            uni.request({
                                url: 'http://localhost:8088/user/getBalance?id=' + res.data,
                                method: 'get',
                                success: (res) => {
									this.balance = res.data;
                                    uni.setStorageSync('balance', res.data);
                                }
                            });
                        }
                    });
                },
                fail: (res) => {
                    console.log('获取用户个人信息失败: ', res);
                    //用户按了拒绝按钮
                    uni.showModal({
                        title: '警告',
                        content: '您点击了拒绝授权，将无法进入小程序，请授权之后再进入!!!',
                        showCancel: false,
                        confirmText: '返回授权',
                        success: function (res) {
                            // 用户没有授权成功，不需要改变 isHide 的值
                            if (res.confirm) {
                                console.log('用户点击了“返回授权”');
                            }
                        }
                    });
                }
            });
        }
    }
};
</script>
<style>
@import './user.css';
</style>
