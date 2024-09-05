<template>
    <view class="container">
        <view class="canlendarBgView">
            <view class="canlendarView">
                <view class="canlendarTopView">
                    <view class="leftBgView" @tap="handleCalendar" data-handle="prev">
                        <view class="leftView">《</view>
                    </view>
                    <view class="centerView">{{ cur_year || '--' }} 年 {{ cur_month || '--' }} 月</view>
                    <view class="rightBgView" @tap="handleCalendar" data-handle="next">
                        <view class="rightView">》</view>
                    </view>
                </view>
                <view class="weekBgView">
                    <view class="weekView" :data-idx="index" v-for="(item, index) in weeks_ch" :key="index">{{ item }}</view>
                </view>
                <view class="dateBgView">
                    <view v-if="hasEmptyGrid" class="dateEmptyView" :data-idx="index" v-for="(item, index) in empytGrids" :key="index"></view>
                    <view class="dateView" :data-idx="index" @tap="dateSelectAction" v-for="(item, index) in days" :key="item.date">
                        <!-- 如果预订 添加 ordering class -->

                        <view
                            :class="
                                'datesView ' + (index == selectIndex ? 'dateSelectView' : index == todayIndex ? 'todayView' : '') + ' ' + (item.isOrder ? 'ordering' : '') + ' '
                            "
                        >
                            {{ item.date }}
                        </view>
                    </view>
                </view>
            </view>
            <view class="delete">长按可删除预约记录</view>
        </view>

        <view class="ordering-lists">
            <!-- 加列表的时候 添加 animated class -->
            <!-- <view class="ordering-item" bindlongtap="longTap">2019年5月12日 周四</view>
        <view class="ordering-item animated" bindlongtap="longTap">2019年5月12日 周四</view> -->
            <block v-for="(item, idx) in dateList" :key="idx">
                <view class="ordering-item" :data-select-id="item.id" :data-select-date="item.time" :data-tip-date="item.date" @longpress="longTap">{{ item.date }}</view>
            </block>
        </view>
        <view class="btn-add" @tap="addNewReservation">+</view>
    </view>
</template>

<script>
const common = require('../../utils/util');
const app = getApp();
export default {
    data() {
        return {
            hasEmptyGrid: false,
            cur_year: '',
            cur_month: '',

            //选中的那天
            currentDay: '',

            //选中的时间
            dinTime: '',

            //数据
            dateList: [],

            //存放当前月份已预约的日期
            matchDateList: [],

            //存放当前月份已报餐的日期
            allMealRecordList: [],

            saturdayCanDiner: '',
            sundayCanDiner: '',
            days: '',
            nickName: '',
            userDepartmentName: '',
            selectIndex: '',
            weeks_ch: '',
            todayIndex: '',
            empytGrids: ''
        };
    },
    onLoad(options) {
        const that = this;
        that.setNowDate();
    },
    /**
     * 生命周期函数--监听页面显示
     */
    onShow: function () {
        const that = this;
        that.viewAllReservation();
        that.AllMealRecords();
        that.getConfig();
    },
    methods: {
        /**
         * 获取配置信息
         */
        getConfig: function () {
            const that = this;
            uni.request({
                url: app.globalData.web_path + '/bc/' + app.globalData.appId + '/config/getConfig',
                header: app.globalData.header,
                success: function (res) {
                    that.setData({
                        saturdayCanDiner: res.data.data.saturdayCanDiner,
                        sundayCanDiner: res.data.data.sundayCanDiner
                    });
                }
            });
        },

        /**
         * 将list 存放到预约记录 dateList中
         */
        pustData: function (data) {
            const that = this;
            const list = data;
            for (let i = 0; i < list.length; i++) {
                let bcReserveRecordId = list[i].id;
                let reserveTime = new Date(Date.parse(list[i].reserveTime.replace(/-/g, '/')));
                reserveTime = reserveTime.getFullYear() + '年' + (reserveTime.getMonth() + 1) + '月' + reserveTime.getDate() + '日';
                let reserveTimeWeek = list[i].reserveTimeWeek;
                let dateStr = reserveTime + ' ' + reserveTimeWeek;
                let dateArr = {
                    date: dateStr,
                    time: list[i].reserveTime,
                    id: bcReserveRecordId
                };
                that.dateList.push(dateArr);
            }
            that.setData({
                dateList: that.dateList
            });
        },

        /**
         * 查看所有的预约记录
         */
        viewAllReservation: function () {
            const that = this;
            that.dateList = [];
            const Token = uni.getStorageSync('Token');
            if (Token) {
                uni.showLoading({
                    title: '加载中',
                    mask: true
                });
                app.globalData.header.Token = Token;
                uni.request({
                    url: app.globalData.web_path + '/bc/' + app.globalData.appId + '/bcReserveRecord/bcReserveRecordList',
                    header: app.globalData.header,
                    success: (res) => {
                        that.pustData(res.data.data);
                        that.matchDateList = [];
                        that.MatchYearAndMonth();
                        uni.hideLoading();
                    },
                    fail: (res) => {}
                });
            }
        },

        /**
         * 根据当前年份和月份查询当前年月有哪些日期被预约了
         */
        MatchYearAndMonth: function () {
            const that = this;
            const Token = uni.getStorageSync('Token');
            if (Token) {
                const month = that.cur_month < 10 ? '0' + that.cur_month : that.cur_month;
                const curYearMonth = that.cur_year + '-' + month;
                uni.request({
                    url: app.globalData.web_path + '/bc/' + app.globalData.appId + '/bcReserveRecord/getBcReserveRecordListByCurYearAndCurMonth',
                    header: app.globalData.header,
                    data: {
                        curYearMonth: curYearMonth
                    },
                    success: function (res) {
                        let list = res.data.data;
                        for (let i = 0; i < list.length; i++) {
                            let matchReserveTime = list[i].reserveTime;
                            let orderDay = matchReserveTime.substr(8, 2);
                            orderDay = parseInt(orderDay) < 10 ? orderDay.substr(1, 1) : orderDay;
                            that.matchDateList.push(orderDay);
                        }
                        that.setData({
                            matchDateList: that.matchDateList
                        });
                        that.onAppointment();
                    }
                });
            }
        },

        /**
         * 根据当前年份和月份查询当前年月有哪些日期被报餐了
         */
        AllMealRecords: function () {
            const that = this;
            that.allMealRecordList = [];
            const Token = uni.getStorageSync('Token');
            if (Token) {
                let month = that.cur_month < 10 ? '0' + that.cur_month : that.cur_month;
                let dinTime = that.cur_year + '-' + month;
                uni.request({
                    url: app.globalData.web_path + '/bc/' + app.globalData.appId + '/BcRecord/bcBcRecordByMonth',
                    header: app.globalData.header,
                    data: {
                        dinTime: dinTime
                    },
                    success: function (res) {
                        let list = res.data.data;
                        for (let i = 0; i < list.length; i++) {
                            let dinTime = list[i].day;
                            that.allMealRecordList.push(dinTime);
                        }
                        that.setData({
                            allMealRecordList: that.allMealRecordList
                        });
                    },
                    fail: function (res) {}
                });
            }
        },

        /**
         * 匹配预约记录和日历有哪些是被预约的
         */
        onAppointment: function () {
            const that = this;
            const days = uni.getStorageSync('days');
            const orderArr = that.matchDateList;
            for (let i = 0; i < days.length; i++) {
                for (let j = 0; j < orderArr.length; j++) {
                    if (orderArr[j] == days[i].date) {
                        days[i].isOrder = true;
                    }
                }
            }
            that.setData({
                days: days
            });
        },

        /**
         * 点击+号,新增一条记录
         * @param e
         */
        addNewReservation: function (e) {
            const that = this;
            const week = common.getMyDay(new Date(`${that.cur_year}/${that.cur_month}/${that.currentDay + 1}`));
            const Token = uni.getStorageSync('Token');
            const now = new Date();
            const nowYear = now.getFullYear();
            const nowMonth = now.getMonth() + 1;
            const nowYearAndMonth = nowYear + '-' + nowMonth;
            const todayYearAndMonth = that.cur_year + '-' + that.cur_month;
            const noSelectDay = new Date(Date.parse(that.dinTime.replace(/-/g, '/')));
            const selectIndex = that.selectIndex + 1;
            if (Token) {
                app.globalData.header.Token = Token;
                if (that.allMealRecordList.length > 0) {
                    for (let val of that.allMealRecordList) {
                        if (val == selectIndex) {
                            common.showModel('不能预订已报餐的日期');
                            return;
                        }
                    }
                }
                if (that.matchDateList.length > 0) {
                    for (let i = 0; i < that.matchDateList.length; i++) {
                        if (that.matchDateList[i] == selectIndex) {
                            common.showModel('你已经预约了');
                            return;
                        }
                    }
                }
                if (that.selectIndex == that.todayIndex && todayYearAndMonth === nowYearAndMonth) {
                    common.showModel('只能预约比今天大的日期');
                } else if (!that.dinTime) {
                    common.showModel('请选择日期');
                } else if (week == '周六' && that.saturdayCanDiner == false) {
                    common.showModel('星期六不能预约');
                } else if (week == '周日' && that.sundayCanDiner == false) {
                    common.showModel('星期天不能预约');
                } else if (noSelectDay < now) {
                    common.showModel('只能预约比今天大的日期');
                } else {
                    uni.request({
                        url: app.globalData.web_path + '/bc/' + app.globalData.appId + '/bcReserveRecord/bcReserveRecordSave',
                        header: app.globalData.header,
                        data: {
                            reserveTime: that.dinTime,
                            reserveTimeWeek: week
                        },
                        success: function (res) {
                            //返回的结果为1，说明用户未激活
                            if (res.data.code == 2) {
                                common.showModel('中午报餐未开启');
                            } else if (res.data.code == 1) {
                                common.showModel('请联系管理员给予激活');
                            } else {
                                that.viewAllReservation();
                            }
                        },
                        fail: function (res) {}
                    });
                }
            } else {
                uni.getSetting({
                    success(res) {
                        //没有授权,引导用户授权
                        if (!res.authSetting['scope.userInfo']) {
                            //console.log('没有授权,引导用户授权',res)
                            uni.showModal({
                                title: '用户未授权',
                                content: '如需正常使用红商报餐，请按确定并在授权管理中选中“用户信息”，然后点按确定。最后再重新进入小程序即可正常使用。',
                                showCancel: false,
                                success: function (res) {
                                    if (res.confirm) {
                                        uni.openSetting({
                                            success: function success(res) {
                                                console.log('调用openSetting方法success:', res);
                                                uni.getUserInfo({
                                                    success: function (res) {
                                                        console.log('userInfo', res);
                                                        that.saveUserInfo(res.userInfo, res.iv, res.signature, res.encryptedData, res.rawData);
                                                    }
                                                });
                                            },
                                            fail: function (res) {
                                                console.log('调用openSetting方法fail:', res);
                                            }
                                        });
                                    }
                                }
                            });
                        } else {
                            uni.navigateTo({
                                url: '/pages/user-info/index'
                            });
                        }
                    }
                });
            }
        },

        /**
         * 有用户授权信息向后台发送请求
         * @param userInfo
         * @param iv
         * @param signature
         * @param encryptedData
         * @param rawData
         */
        saveUserInfo: function (userInfo, iv, signature, encryptedData, rawData) {
            const that = this;
            uni.login({
                success: function (res) {
                    //如果有状态码就向后台发送请求
                    if (res.code) {
                        uni.request({
                            url: app.globalData.web_path + '/wx/user/' + app.globalData.appId + '/login',
                            data: {
                                code: res.code,
                                loginType: 'openid'
                            },
                            header: app.globalData.header,
                            success: function (res) {
                                //将openid  缓存
                                uni.setStorageSync('openid', res.data.data.openid);
                                uni.setStorageSync('sessionKey', res.data.data.sessionKey);
                                //判断是否存在token,不存在则说明该用户没有注册,需要弹出用户登录页面,有则不需要进行页面跳转
                                if (!res.data.data.Token && userInfo) {
                                    //保存用户信息
                                    uni.request({
                                        url: app.globalData.web_path + '/wx/user/' + app.globalData.appId + '/info',
                                        data: {
                                            sessionKey: res.data.data.sessionKey,
                                            iv: iv,
                                            signature: signature,
                                            encryptedData: encryptedData,
                                            rawData: rawData
                                        },
                                        header: app.globalData.header,
                                        success: function (res) {
                                            uni.showModal({
                                                content: '为确保红商集团报餐小程序更好的使用，请完善个人信息',
                                                showCancel: false,
                                                success: function (res) {
                                                    if (res.confirm) {
                                                        let imgUrl = userInfo.avatarUrl;
                                                        uni.setStorageSync('imgUrl', imgUrl);
                                                        uni.navigateTo({
                                                            url: '/pages/user-info/index'
                                                        });
                                                    }
                                                }
                                            });
                                        },
                                        fail: function (res) {}
                                    });
                                } else {
                                    //放在请求头里
                                    app.globalData.header.Token = res.data.data.Token;
                                    uni.setStorageSync('Token', res.data.data.Token);
                                    uni.setStorageSync('user', res.data.data.user);
                                    uni.setStorageSync('userDepartmentName', res.data.data.userDepartmentName);
                                    var name = uni.getStorageSync('user').name;
                                    var deptName = uni.getStorageSync('userDepartmentName');
                                    if (name) {
                                        that.setData({
                                            nickName: name
                                        });
                                    }
                                    if (deptName) {
                                        that.setData({
                                            userDepartmentName: deptName
                                        });
                                    }
                                }
                            },
                            error: function (res) {
                                console.log('-', '请求失败');
                            }
                        });
                    }
                },
                fail: function (res) {
                    console.log('error', '接口调用失败!');
                }
            });
        },

        /**
         * 选择日期
         * @param e
         */
        dateSelectAction: function (e) {
            const that = this;
            let cur_day = e.currentTarget.dataset.idx;
            let month = that.cur_month < 10 ? '0' + that.cur_month : that.cur_month;
            let day = cur_day + 1 < 10 ? '0' + (cur_day + 1) : cur_day + 1;
            let dinTime = that.cur_year + '-' + month + '-' + day;
            that.setData({
                dinTime: dinTime,
                selectIndex: cur_day,
                currentDay: cur_day
            });
        },

        /**
         * 设置当前时间
         */
        setNowDate: function () {
            const date = new Date();
            const cur_year = date.getFullYear();
            const cur_month = date.getMonth() + 1;
            const todayIndex = date.getDate() - 1;
            const selectIndex = date.getDate() - 1;
            const weeks_ch = ['日', '一', '二', '三', '四', '五', '六'];
            this.calculateEmptyGrids(cur_year, cur_month);
            this.calculateDays(cur_year, cur_month);
            this.setData({
                cur_year: cur_year,
                cur_month: cur_month,
                weeks_ch,
                todayIndex,
                selectIndex
            });
        },

        /**
         * 表示当前月份的前一天，就是year/month/30这天
         * @param year
         * @param month
         * @returns {number}
         */
        getThisMonthDays(year, month) {
            return new Date(year, month, 0).getDate();
        },

        /**
         * 获取当月的第一周第一天是周几
         * @param year
         * @param month
         * @returns {number}
         */
        getFirstDayOfWeek(year, month) {
            return new Date(Date.UTC(year, month - 1, 1)).getDay();
        },

        /**
         * 计算当前年月空的几天
         * @param year
         * @param month
         */
        calculateEmptyGrids(year, month) {
            const firstDayOfWeek = this.getFirstDayOfWeek(year, month);
            let empytGrids = [];
            if (firstDayOfWeek > 0) {
                for (let i = 0; i < firstDayOfWeek; i++) {
                    empytGrids.push(i);
                }
                this.setData({
                    hasEmptyGrid: true,
                    empytGrids
                });
            } else {
                this.setData({
                    hasEmptyGrid: false,
                    empytGrids: []
                });
            }
        },

        /**
         * 计算当前月份的天数
         * @param year
         * @param month
         */
        calculateDays(year, month) {
            let days = [];
            const thisMonthDays = this.getThisMonthDays(year, month);
            for (let i = 1; i <= thisMonthDays; i++) {
                let obj = {
                    date: i,
                    isOrder: false
                };
                days.push(obj);
            }
            this.setData({
                days
            });
            uni.setStorageSync('days', days);
        },

        /**
         * 切换日历
         * @param e
         */
        handleCalendar(e) {
            let that = this;
            const handle = e.currentTarget.dataset.handle;
            const cur_year = that.cur_year;
            const cur_month = that.cur_month;
            if (handle === 'prev') {
                let newMonth = cur_month - 1;
                let newYear = cur_year;
                if (newMonth < 1) {
                    newYear = cur_year - 1;
                    newMonth = 12;
                }
                that.calculateDays(newYear, newMonth);
                that.calculateEmptyGrids(newYear, newMonth);
                that.setData({
                    cur_year: newYear,
                    cur_month: newMonth,
                    selectIndex: null,
                    dinTime: ''
                });
            } else {
                let newMonth = cur_month + 1;
                let newYear = cur_year;
                if (newMonth > 12) {
                    newYear = cur_year + 1;
                    newMonth = 1;
                }
                that.calculateDays(newYear, newMonth);
                that.calculateEmptyGrids(newYear, newMonth);
                that.setData({
                    cur_year: newYear,
                    cur_month: newMonth,
                    selectIndex: null,
                    dinTime: ''
                });
            }
            if (that.matchDateList.length > 0) {
                that.matchDateList = [];
            }
            that.MatchYearAndMonth();
            if (that.allMealRecordList.length > 0) {
                that.allMealRecordList = [];
            }
            that.AllMealRecords();
        },

        /**
         * 撤销已选择的日期
         * @param event
         */
        longTap(event) {
            var that = this;
            common.showConfirm('撤销报餐', '确认撤销' + event.currentTarget.dataset.tipDate + '报餐吗？').then((resolve) =>
                uni.request({
                    url: app.globalData.web_path + '/bc/' + app.globalData.appId + '/bcReserveRecord/deleteBcReserveRecord',
                    header: app.globalData.header,
                    data: {
                        id: event.currentTarget.dataset.selectId
                    },
                    success: function (res) {
                        if (res.data.data.delResult != 0) {
                            that.viewAllReservation();
                        }
                    },
                    fail: function (res) {}
                })
            );
        }
    }
};
</script>
<style>
.canlendarBgView {
    display: flex;
    flex-grow: 1;
    flex-direction: column;
    align-items: center;
    position: fixed;
    top: 0;
    background: white;
    box-shadow: 0 10px 10px rgb(255, 255, 255);
    z-index: 1;
}
.canlendarView {
    display: flex;
    color: #47a7dd;
    flex-direction: column;
}
.canlendarTopView {
    display: flex;
    height: 80rpx;
    font-size: 32rpx;
    flex-direction: row;
    align-items: center;
    justify-content: center;
    font-weight: 600;
}
.leftBgView {
    display: flex;
    height: 80rpx;
    text-align: right;
    flex-direction: row-reverse;
}
.leftView {
    display: flex;
    width: 80rpx;
    height: 100%;
    flex-direction: row;
    align-items: center;
    justify-content: center;
}
.centerView {
    display: flex;
    width: 50%;
    height: 80rpx;
    text-align: center;
    flex-direction: row;
    align-items: center;
    justify-content: center;
}
.rightBgView {
    display: flex;
    height: 80rpx;
    flex-direction: row;
}
.rightView {
    display: flex;
    width: 80rpx;
    height: 100%;
    flex-direction: row;
    align-items: center;
    justify-content: center;
}
.weekBgView {
    display: flex;
    height: 50rpx;
    line-height: 50rpx;
    opacity: 0.5;
    flex-direction: row;
    justify-content: center;
    align-items: center;
}
.weekView {
    text-align: center;
    font-size: 28rpx;
    flex-grow: 1;
}
.dateBgView {
    display: flex;
    height: 500rpx;
    flex-direction: row;
    flex-wrap: wrap;
}
.dateEmptyView {
    display: flex;
    width: 107.1428571429rpx;
    color: #fff;
    align-items: center;
    justify-content: center;
}
.dateView {
    display: flex;
    width: 107.1428571429rpx;
    color: #fff;
    align-items: center;
    justify-content: center;
}
.datesView {
    display: flex;
    width: 60rpx;
    height: 60rpx;
    color: #47a7dd;
    font-size: 30rpx;
    align-items: center;
    justify-content: center;
    line-height: 1;
}
.todayView,
.dateSelectView,
.ordering {
    position: relative;
    top: 0;
    left: 0;
    border-radius: 50%;
}
.todayView {
    background: #eee;
}
.dateSelectView {
    background-color: #1c7add;
    color: #fff;
}
.ordering {
    position: relative;
    top: 0;
    left: 0;
    border-radius: 50%;
    background-color: #df2a47;
    color: #fff;
}

.ordering-lists {
    padding: 370px 24rpx 30rpx;
    min-height: 100vh;
    box-sizing: border-box;
}
.ordering-item {
    background: white;
    border-radius: 6rpx;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
    padding: 0 40rpx;
    height: 100rpx;
    line-height: 100rpx;
    font-size: 32rpx;
    color: #333;
    position: relative;
    margin-bottom: 30rpx;
    transition: all 0.3s;
}
.ordering-item.animated {
    animation: translate 0.4s;
}
@keyframes translate {
    from {
        transform: translateY(80%);
    }
    to {
        transform: translateY(0%);
    }
}
.ordering-item::before {
    content: '';
    position: absolute;
    left: 0;
    top: 0;
    bottom: 0;
    width: 12rpx;
    border-top-left-radius: 10rpx;
    border-bottom-left-radius: 10rpx;
    background: #ff7e00;
}

.btn-add {
    width: 90rpx;
    height: 90rpx;
    line-height: 90rpx;
    border-radius: 50%;
    background: #dd3e41;
    color: white;
    font-size: 30px;
    text-align: center;
    position: fixed;
    bottom: 18vh;
    right: 40rpx;
}
.delete {
    font-size: 30rpx;
    margin-top: 20rpx;
    color: #ff7e00;
}
</style>
