<template>
    <view>
        <view class="canvas-conter">
            <view class="tabs flex">
                <block v-for="(item, index) in tabs" :key="index">
                    <view :class="'tab ' + (index == curIndex ? 'tab--active' : '')" @tap="tabClick" :id="index">{{ item }}</view>
                </block>
            </view>
            <ec-canvas id="mychart-dom-pie" canvas-id="mychart-pie" :ec="ec"></ec-canvas>
            <cover-view class="pie-text">
                <cover-view class="pie-subtitle">总报餐人数</cover-view>
                <cover-view class="pie-title">{{ totalNum }}</cover-view>
            </cover-view>
            <cover-view class="echart-legend">
                <block v-for="(item, index) in deptList" :key="item.id">
                    <cover-view :class="'echart-legend__item ' + (index == selectIndex ? 'checked' : '')">
                        <cover-view class="echart-legend__item-pageIcon" />
                        <cover-view class="echart-legend__item-text" :data-id="item.id" :data-index="index" @tap="selectDept">{{ item.text }}</cover-view>
                    </cover-view>
                </block>
            </cover-view>
        </view>

        <view class="table">
            <view class="th flex">
                <view class="td flex-item">报餐详情</view>
            </view>
            <block v-if="bcRecordList.length != 0">
                <view class="tr flex" :data-id="item.id" :data-name="item.name" :data-index="index" @tap="hadEatFun" v-for="(item, index) in bcRecordList" :key="index">
                    <view class="td flex">
                        <image class="avatar" :src="item.avatarurl" lazy-load />
                        <image v-if="item.hadEat == 1" class="icon-checked" src="/static/images/icon/icon-check.png" />
                    </view>

                    <view class="td flex-item">
                        <view class="name">{{ item.name }}</view>
                        <view class="time">{{ item.addTime }}</view>
                    </view>

                    <view class="td">
                        <view class="department">{{ item.deptName }}</view>
                        <view :class="'type ' + (item.channel == '手动报餐' ? 'yellowish' : 'green')">{{ item.channel }}</view>
                    </view>
                </view>
            </block>
            <block v-else>
                <view class="tr">
                    <view class="td flex-item">
                        <image class="empty-img" src="/static/images/icon/icon-empty.png" />
                        <view class="empty-text">{{ emptyText }}</view>
                    </view>
                </view>
            </block>
        </view>
    </view>
</template>

<script>
import ecCanvas from '@/ec-canvas/ec-canvas';
import * as echarts from '../../ec-canvas/echarts';
const app = getApp();
let chart = null; //定义变量接收echarts实例,方便改数据
export default {
    components: {
        ecCanvas
    },
    //页面加载
    data() {
        return {
            menuTop: '',

            ec: {
                lazyLoad: true
            },

            tabs: ['今天', '明天'],
            curIndex: 0,
            totalNum: '0',
            bcRecordList: [],
            deptList: [],
            currentPage: 1,
            pageSize: 10,
            emptyText: '该日暂无员工报餐',
            hadEat: false,
            deptId: '',
            selectIndex: ''
        };
    },
    onLoad: function (options) {
        // 获取组件
        this.ecComponent = this.zpSelectComponent('#mychart-dom-pie');
    },
    // 菜单栏吸顶
    onShow: function () {
        var that = this;
        uni.removeStorageSync('datas');
        that.setData({
            currentPage: 1,
            bcRecordList: [],
            deptId: '',
            selectIndex: null
        });
        that.initChart(); //初始化环形图
        that.getTotalNum(); //查询报餐人数(默认查询当天的)
        that.getBcRecordList(); // 查询报餐列表（默认查询当天的）
    },
    methods: {
        // tab切换
        tabClick(e) {
            var that = this;
            uni.removeStorageSync('datas');
            that.setData({
                curIndex: e.currentTarget.id,
                bcRecordList: [],
                currentPage: 1,
                deptId: '',
                selectIndex: null
            });
            chart.clear(); //清空实例,防止数字闪烁
            that.initChart();
            that.getTotalNum(); //查询报餐人数
            that.getBcRecordList(); // 查询报餐列表
        },

        // e-chart 环形图
        initChart: function () {
            const that = this;
            // 获取组件的 canvas、width、height 后的回调函数
            that.ecComponent.init((canvas, width, height) => {
                // 在这里初始化图表
                chart = echarts.init(canvas, null, {
                    width: width,
                    height: height
                });
                that.setOption(chart);
                // 注意这里一定要返回 chart 实例，否则会影响事件处理等
                return chart;
            });
        },

        selectDept() {
            console.log('占位：函数 selectDept 未声明');
        },

        hadEatFun() {
            console.log('占位：函数 hadEat 未声明');
        }
    }
};
</script>
<style>
page {
    min-height: 100vh;
    padding-bottom: 40rpx;
    padding-top: 20rpx;
    box-sizing: border-box;
}
.canvas-conter {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    box-sizing: border-box;
    width: 702rpx;
    border-radius: 10rpx;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
    background: white;
    margin: 0 auto 10px;
    position: relative;
}
.tabs {
    width: 140px;
    height: 54rpx;
    margin: 50rpx auto 0;
}
.tabs .tab {
    width: 70px;
    height: 28px;
    border: 1px solid #d7242d;
    box-sizing: border-box;
    color: #d7242d;
    text-align: center;
    line-height: 26px;
    font-size: 26rpx;
}
.tabs .tab:first-child {
    border-top-left-radius: 4px;
    border-bottom-left-radius: 4px;
    border-right: 0;
}
.tabs .tab:last-child {
    border-top-right-radius: 4px;
    border-bottom-right-radius: 4px;
    border-left: 0;
}
.tabs .tab--active {
    color: white;
    background: #d7242d;
}
ec-canvas {
    width: 700rpx;
    height: 400rpx;
}
.pie-text {
    position: absolute;
    left: 33%;
    bottom: 0;
    width: 150px;
    height: 400rpx;
    display: flex;
    justify-content: center;
    text-align: center;
    flex-direction: column;
    margin-left: -75px;
}
.pie-subtitle {
    font-size: 24rpx;
}
.pie-title {
    font-size: 34rpx;
    margin-top: 10rpx;
}

.echart-legend {
    position: absolute;
    right: 56rpx;
    top: 50%;
    transform: translateY(-25%);
}
.echart-legend__item {
    display: flex;
    align-items: center;
    margin: 16rpx 0;
}
.echart-legend__item-pageIcon {
    width: 28rpx;
    height: 28rpx;
    border-radius: 50%;
    margin-right: 18rpx;
}
.echart-legend__item:nth-child(1) .echart-legend__item-pageIcon {
    background: #fe5d4d;
}
.echart-legend__item:nth-child(2) .echart-legend__item-pageIcon {
    background: #737dde;
}
.echart-legend__item:nth-child(3) .echart-legend__item-pageIcon {
    background: #3ba4ff;
}
.echart-legend__item:nth-child(4) .echart-legend__item-pageIcon {
    background: #fe9e7f;
}
.echart-legend__item:nth-child(5) .echart-legend__item-pageIcon {
    background: #feda5e;
}
.echart-legend__item:nth-child(6) .echart-legend__item-pageIcon {
    background: #66dfe2;
}
.echart-legend__item:nth-child(7) .echart-legend__item-pageIcon {
    background: #37a1d9;
}
.echart-legend__item-text {
    font-size: 13px;
    color: #999999;
}
.checked .echart-legend__item-text {
    color: #333;
    font-weight: 600;
}

.table {
    padding: 30rpx 30rpx 40rpx;
    font-size: 30rpx;
}
.table .th {
    color: #666;
    padding: 24rpx 0;
    background: white;
}
.table .tr {
    padding: 24rpx 10rpx;
    position: relative;
    display: flex;
    align-items: center;
}
.table .tr::before {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    height: 1px;
    transform: scaleY(0.5);
    background: #e5e5e5;
}
.table .tr .td:last-child {
    margin-left: 20rpx;
}

.table .td {
    position: relative;
}
.table .avatar {
    width: 110rpx;
    height: 110rpx;
    border-radius: 50%;
    margin-right: 20rpx;
}
.table .icon-checked {
    width: 50rpx;
    height: 50rpx;
    position: absolute;
    bottom: -8rpx;
    right: 6rpx;
}
.table .name {
    font-size: 30rpx;
}
.table .time {
    margin-top: 1px;
    font-size: 26rpx;
    color: #666;
}
.table .department {
    text-align: right;
    font-size: 28rpx;
}
.table .type {
    margin-top: 1px;
    font-size: 24rpx;
    color: #999;
    text-align: right;
}
.table .type.green {
    color: #07c160;
}
.table .type.yellowish {
    color: #ff7e00;
}

.empty-img {
    width: 270rpx;
    height: 170rpx;
    display: block;
    margin: 60rpx auto 0;
}
.empty-text {
    text-align: center;
    margin-top: 30rpx;
    font-size: 26rpx;
    color: #666;
}
</style>
