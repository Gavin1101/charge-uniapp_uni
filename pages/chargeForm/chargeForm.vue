<template>
    <view class="page">
        <div id="form">
            <van-cell-group>
                <van-field :value="formData.name" @change="formNameChange" name="name" label="名称" :rules="[{ required: true,message:'请填写名称'}]">
                </van-field>
                <van-field :value="formData.address" @change="formAddressChange" label="地址" :rules="[{ required: true,message:'请填写地址'}]">
                </van-field>
                <van-field :value="formData.price" @change="formPriceChange" label="价格" :rules="[{ required: true,message:'请填写价格'}]">
                </van-field>
                <van-field :value="formData.sum" @change="formSumChange" label="数量" :rules="[{ required: true,message:'请填写数量'}]">
                </van-field>
                <van-field :value="formData.num" @change="formNumChange" label="剩余数量" :rules="[{ required: true,message:'请填写剩余数量'}]">
                </van-field>
                <van-field label="类型">
					<template #input>
						<van-radio-group @change="formRadioChange" :value="formData.type" direction="horizontal">
							<van-radio name="1">快充</van-radio>
							<van-radio name="0">慢充</van-radio>
						</van-radio-group>
					</template>
                </van-field>
            </van-cell-group>
        </div>
        <view class="weui-btn-area">
            <button class="weui-btn" type="primary" @tap="submitForm">提交</button>
        </view>
    </view>
</template>

<script>
// pages/chargeForm/chargeForm.js
export default {
    components: {},
    data() {
        return {
            userId: '',
            userName: '',
            formData: {
                name: '',
                address: '',
                price: '',
                sum: '',
                num: '',
                type: ''
            },

            rules: []
        };
    },
    /**
     * 生命周期函数--监听页面加载
     */
    onLoad(options) {
        this.userId = uni.getStorageSync("userId")
		this.userName = uni.getStorageSync("username")
        if (options.data) {
			this.formData=JSON.parse(options.data);
			console.log("this.formData")
        }
		console.log(this.formData)
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
    onUnload() {
        uni.navigateTo({
            url: '/pages/myCharge/myCharge'
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
		formNameChange(e) {
			console.log(e.detail)
			this.formData.name=e.detail
		},
		formAddressChange(e) {
			this.formData.address=e.detail
		},
		formPriceChange(e) {
			this.formData.price=e.detail
		},
		formSumChange(e) {
			this.formData.sum=e.detail
		},
		formNumChange(e) {
			this.formData.num=e.detail
		},
		formRadioChange(e){
			console.log(e)
			this.formData.type=e.detail
		},
        submitForm() {
            if (this.formData.id) {
                this.formData.state = '1';
                this.formData.owner = this.userName;
                this.formData.userId = this.userId;
                uni.request({
                    url: 'http://localhost:8088/charge/updateCharge',
                    method: 'post',
                    data: this.formData,
                    success: (res) => {
                        console.log(res.data);
                        if (res.data == '' || res.data == null) {
                            uni.showToast({
                                title: '地址错误',
                                icon: 'error'
                            });
                        } else {
                            uni.navigateTo({
                                url: '/pages/myCharge/myCharge'
                            });
                        }
                    }
                });
            } else {
                this.formData.state = '1';
                this.formData.owner = this.userName;
                this.formData.userId = this.userId;
				console.log(this.formData)
                uni.request({
                    url: 'http://localhost:8088/charge/chargeAdd',
                    method: 'post',
                    data: this.formData,
                    success: (res) => {
                        console.log(res.data);
                        if (res.data == '' || res.data == null) {
                            uni.showToast({
                                title: '地址错误',
                                icon: 'error'
                            });
                        } else {
                            uni.navigateTo({
                                url: '/pages/myCharge/myCharge'
                            });
                        }
                    }
                });
            }
        }
    }
};
</script>
<style>
@import './chargeForm.css';
</style>
