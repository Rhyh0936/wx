<template>
    <view>
        <view class="container">
            <view class="header flex">
                <view class="user-info flex-item">
                    <text>{{ timeSlot }},</text>
                    <text>{{ nickName }}</text>
                    <text class="user-branch">{{ userDepartmentName }}</text>
                </view>
                <view class="today flex">
                    <image class="icon-clander" src="/static/images/icon/icon-clander.png" lazy-load mode="widthFix" />
                    <text>{{ cur_year }}年{{ cur_month }}月{{ cur_day }}日</text>
                </view>
            </view>

            <swiper class="swiper" indicator-color="rgba(255,255,255,0.3)" indicator-active-color="#ffffff" :autoplay="true" :circular="true" interval="5000" duration="1000">
                <block v-for="(item, idx) in imgUrlList" :key="idx">
                    <swiper-item>
                        <image :src="item.imgUrl" mode="aspectFill" @tap="clickSwiper" :data-url="item.imgLink" />
                    </swiper-item>
                </block>
            </swiper>

            <view class="btn-wrap">
                <button :class="'flex  ' + (select == 0 ? 'btn-meal' : 'cbtn-meal')" type="primary" :disabled="disabled" @tap="openToast">
                    <image src="/static/images/icon/icon-tableware.png" mode="widthFix" />
                    <view ref="choose">
                        <view class="btn-title">{{ mealOffer }}</view>
                        <view class="btn-subtitle">Began To Eat</view>
                    </view>
                </button>
            </view>

            <view class="count-down flex">
                <view class="count-down__title">报 餐 倒 计 时</view>
                <!-- <view class="count-down__content">01 : 50 : 23</view> -->
                <view class="count-down__content">{{ clock }}</view>
                <view class="count-down__desc">
                    <text>H</text>
                    <text>M</text>
                    <text>S</text>
                </view>
            </view>
        </view>

        <!-- 授权弹窗 -->
        <view class="popup empower" v-if="isShowEmpower">
            <view class="mask"></view>
            <view :class="'ep-content popup-box ' + (animated ? 'animated' : '')">
                <view class="ep-tit t-center">需要您的授权</view>
                <text class="ep-subtit t-center">为了提供更好的服务\n请在稍后的提示框中点击“允许”</text>
                <image class="ep-img" src="http://img.rblcmall.com/wxapp/images/empower.png"></image>
                <button class="btn btn-primary btn-confirm" open-type="getUserInfo" lang="zh_CN" @getuserinfo="clickEmpower">我知道了</button>
            </view>
        </view>
    </view>
</template>

<script>
const app = getApp();
const common = require('../../utils/util');
export default {
    data() {
        return {
            cur_year: '2019',
            cur_month: '01',
            cur_day: '01',
            //显示授权
            isShowEmpower: false,
            animated: false,
            nickName: '智慧报餐',
            cur_hour: '00',
            cur_minute: '00',
            cur_second: '00',
            timeSlot: '早上好',
            clock: '',
            mealOffer: '开始报餐',
            userDepartmentName: '',
            //报餐状态 0：取消报餐 1：已报餐
            baoCan: 0,
            baoCanRecordId: '',
            //1：就餐时间在明天 2：就餐时间在今天
            orderMeal: 1,
            //截止时间状态
            deadlineStatus: '',
            saturdayCanDiner: false,
            sundayCanDiner: false,
            //禁用状态
            disabled: false,
            //0：未选中开始报餐的样式 1：选中已报餐的样式
            select: 0,
            //定时器任务
            timer: '',
            //图片list集合
            imgUrlList: []
        };
    }
    /**
     * 生命周期函数--监听页面加载
     */,
    onLoad: function (options) {
        this.setNowDate();
        this.isAuthorize();
    },
    /**
     * 生命周期函数--监听页面显示
     */
    onShow: function () {
        const that = this;
        that.searchImgList();
        that.getEndTime();
        //获取截止时间
        const name = uni.getStorageSync('nickName');
        if (name) {
            that.setData({
                nickName: name
            });
        }
        //延时加载时因为获取截止时间 获取慢
        // setTimeout(() => {
        //     that.compareTime();
        // }, 300);
        setTimeout(() => {
            that.countDown(that);
        }, 300);
        setTimeout(() => {
            that.valiBcRecord();
        }, 400);
        const userDepartmentName = uni.getStorageSync('userDepartmentName');
        if (userDepartmentName) {
            that.setData({
                userDepartmentName: userDepartmentName
            });
        }
    },
    onHide: function () {
        let that = this;
        clearInterval(that.timer);
    },
    methods: {
       

        /**
         * 设置当前时间
         */
        setNowDate: function () {
            let that = this;
            const date = new Date();
            const cur_year = date.getFullYear();
            const cur_month = date.getMonth() + 1;
            const cur_day = date.getDate();
            const cur_hour = date.getHours();
            const cur_minute = date.getMinutes();
            const cur_second = date.getSeconds();
            that.setData({
                cur_year: cur_year,
                cur_month: cur_month,
                cur_day: cur_day,
                cur_hour: cur_hour,
                cur_minute: cur_minute,
                cur_second: cur_second
            });
            if (parseInt(that.cur_hour) > 6 && parseInt(that.cur_hour) < 12) {
                that.setData({
                    timeSlot: '早上好'
                });
            } else if (parseInt(that.cur_hour) >= 12 && parseInt(that.cur_hour) <= 13) {
                that.setData({
                    timeSlot: '中午好'
                });
            } else if (parseInt(that.cur_hour) > 13 && parseInt(that.cur_hour) < 18) {
                that.setData({
                    timeSlot: '下午好'
                });
            } else if (parseInt(that.cur_hour) >= 18 && parseInt(that.cur_hour) <= 24) {
                that.setData({
                    timeSlot: '晚上好'
                });
            } else {
                that.setData({
                    timeSlot: '凌晨好'
                });
            }
        },

        clickSwiper() {
            console.log('占位：函数 clickSwiper 未声明');
        },

        clickEmpower() {
            console.log('占位：函数 clickEmpower 未声明');
        }
    }
};
</script>

<style>
/* .container{padding: 24rpx;min-height: 100vh;box-sizing: border-box;} */
.header {
    padding: 30rpx 24rpx 46rpx;
}
.user-info {
    font-size: 30rpx;
}
.user-info .user-branch {
    font-size: 24rpx;
    color: #999;
    margin-left: 8px;
}
.today {
    font-size: 26rpx;
    color: #999;
}
.icon-clander {
    width: 30rpx;
    margin-right: 14rpx;
}

.swiper {
    width: 702rpx;
    height: 300rpx;
    border-radius: 10rpx;
    overflow: hidden;
    margin: 0 auto;
}
.swiper image {
    width: 100%;
    height: 100%;
}

.btn-wrap {
    padding: 66rpx 24rpx;
}
.btn-meal {
    justify-content: center;
    align-items: center;
    background: #d7242d !important;
    line-height: 1;
    height: 100rpx;
    border-radius: 90rpx;
    box-shadow: 0 4px 12px rgba(215, 36, 45, 0.6);
}
.btn-meal::after {
    border: 0 !important;
}
.btn-meal image {
    width: 32rpx;
    margin-right: 20rpx;
}
button.btn-meal[disabled] image,
.button-hover image {
    opacity: 0.5;
}
button.btn-meal[disabled] {
    opacity: 0.8;
}

.cbtn-meal {
    justify-content: center;
    align-items: center;
    background: #d7242d !important;
    line-height: 1;
    height: 100rpx;
    border-radius: 90rpx;
    box-shadow: 0 4px 12px rgba(215, 36, 45, 0.6);
    opacity: 0.6;
}
.cbtn-meal::after {
    border: 0 !important;
}
.cbtn-meal image {
    width: 32rpx;
    margin-right: 20rpx;
}

.btn-title {
    font-size: 32rpx;
}
.btn-subtitle {
    font-size: 20rpx;
    margin-top: 2px;
}

.count-down {
    flex-direction: column;
    align-items: center;
    padding: 30rpx 0;
}
.count-down__title {
    position: relative;
    color: #999999;
    font-size: 34rpx;
}
.count-down__title::after,
.count-down__title::before {
    content: '';
    position: absolute;
    top: 24rpx;
    height: 1px;
    width: 120rpx;
    background: #999;
}
.count-down__title::before {
    left: -160rpx;
}
.count-down__title::after {
    right: -160rpx;
}
.count-down__content {
    font-size: 100rpx;
    color: #ff7e00;
    line-height: 1;
    padding: 50rpx 0 30rpx;
    font-family: Arial !important;
}
.count-down__desc text {
    font-size: 34rpx;
    color: #cccccc;
    padding: 0 82rpx;
}

/* 授权弹窗 */
.ep-content {
    width: 600rpx;
    height: 770rpx;
    padding: 40rpx 50rpx;
    box-sizing: border-box;
    background: white;
    border-radius: 10rpx;
    text-align: center;
}
.ep-tit {
    font-size: 32rpx;
    color: black;
    margin: 30rpx 0;
    line-height: 1;
}
.ep-subtit {
    font-size: 28rpx;
    color: #666;
    margin-bottom: 20rpx;
    display: block;
}
.ep-img {
    width: 482rpx;
    height: 373rpx;
    margin: 0 auto 16rpx;
    display: block;
}
.btn-primary {
    background: #d7242d;
    color: white;
    border-radius: 46px;
}
</style>
