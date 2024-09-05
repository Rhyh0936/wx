<script>
//app.js
export default {
    data() {
        return {};
    },
    globalData: {
        web_path: '服务地址',
        header: {
            Token: '',
            Cookie: '',
            'content-type': 'application/x-www-form-urlencoded'
        },
        appId: 'wx60028a6bb16f015a'
    },
    onLaunch: function () {
        // 展示本地存储能力
        var logs = uni.getStorageSync('logs') || [];
        logs.unshift(Date.now());
        uni.setStorageSync('logs', logs);

        // 登录
        uni.login({
            success: (res) => {
                // 发送 res.code 到后台换取 openId, sessionKey, unionId
            }
        });
        // 获取用户信息
        uni.getSetting({
            success: (res) => {
                if (res.authSetting['scope.userInfo']) {
                    // 已经授权，可以直接调用 getUserInfo 获取头像昵称，不会弹框
                    uni.getUserInfo({
                        success: (res) => {
                            // 可以将 res 发送给后台解码出 unionId
                            this.globalData.userInfo = res.userInfo;

                            // 由于 getUserInfo 是网络请求，可能会在 Page.onLoad 之后才返回
                            // 所以此处加入 callback 以防止这种情况
                            if (this.userInfoReadyCallback) {
                                this.userInfoReadyCallback(res);
                            }
                        }
                    });
                }
            }
        });
    }
};
</script>
<style>
.flex {
    display: flex;
    align-items: center;
}
.flex-item {
    flex: 1;
}

.popup {
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
}
.mask {
    width: 100vw;
    height: 100vh;
    background: rgba(0, 0, 0, 0.7);
}
.popup-box {
    position: fixed;
    top: 50%;
    left: 0;
    right: 0;
    margin: 0 auto;
    transform: translateY(100%);
    transition: all 0.3s;
}
.popup-box.animated {
    transform: translateY(-50%);
}
</style>
