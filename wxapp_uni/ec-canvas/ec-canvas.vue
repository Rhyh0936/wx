<template>
    <view>
        <canvas
            class="ec-canvas"
            :canvas-id="canvasId"
            @init="init"
            @touchstart="parseEventDynamicCode($event, ec.disableTouch ? '' : 'touchStart')"
            @touchmove="parseEventDynamicCode($event, ec.disableTouch ? '' : 'touchMove')"
            @touchend="parseEventDynamicCode($event, ec.disableTouch ? '' : 'touchEnd')"
        ></canvas>
    </view>
</template>

<script>
import WxCanvas from './wx-canvas';
import * as echarts from './echarts';
let ctx;
export default {
    data() {
        return {};
    },
    props: {
        canvasId: {
            type: String,
            default: 'ec-canvas'
        },
        ec: {
            type: Object
        }
    },
    mounted() {
        // 处理小程序 ready 生命周期
        this.$nextTick(() => this.ready());
    },
    methods: {
        ready: function () {
            if (!this.ec) {
                console.warn('组件需绑定 ec 变量，例：<ec-canvas id="mychart-dom-bar" canvas-id="mychart-bar" ec="{{ ec }}"></ec-canvas>');
                return;
            }
            if (!this.ec.lazyLoad) {
                this.init();
            }
        },

        init: function (callback) {
            const version = uni.version.version.split('.').map((n) => parseInt(n, 10));
            const isValid = version[0] > 1 || (version[0] === 1 && version[1] > 9) || (version[0] === 1 && version[1] === 9 && version[2] >= 91);
            if (!isValid) {
                console.error(
                    '\u5FAE\u4FE1\u57FA\u7840\u5E93\u7248\u672C\u8FC7\u4F4E\uFF0C\u9700\u5927\u4E8E\u7B49\u4E8E 1.9.91\u3002\u53C2\u89C1\uFF1Ahttps://github.com/ecomfe/echarts-for-weixin#%E5%BE%AE%E4%BF%A1%E7%89%88%E6%9C%AC%E8%A6%81%E6%B1%82'
                );
                return;
            }
            ctx = uni.createCanvasContext(this.canvasId, this);
            const canvas = new WxCanvas(ctx, this.canvasId);
            echarts.setCanvasCreator(() => {
                return canvas;
            });
            var query = uni.createSelectorQuery().in(this);
            query
                .select('.ec-canvas')
                .boundingClientRect((res) => {
                    if (typeof callback === 'function') {
                        this.chart = callback(canvas, res.width, res.height);
                    } else if (this.ec && typeof this.ec.onInit === 'function') {
                        this.chart = this.ec.onInit(canvas, res.width, res.height);
                    } else {
                        this.$emit('init', {
                            detail: {
                                canvas: canvas,
                                width: res.width,
                                height: res.height
                            }
                        });
                    }
                })
                .exec();
        },

        canvasToTempFilePath(opt) {
            if (!opt.canvasId) {
                opt.canvasId = this.canvasId;
            }
            ctx.draw(true, () => {
                uni.canvasToTempFilePath(opt, this);
            });
        },

        touchStart(e) {
            if (this.chart && e.touches.length > 0) {
                var touch = e.touches[0];
                var handler = this.chart.getZr().handler;
                handler.dispatch('mousedown', {
                    zrX: touch.x,
                    zrY: touch.y
                });
                handler.dispatch('mousemove', {
                    zrX: touch.x,
                    zrY: touch.y
                });
                handler.processGesture(wrapTouch(e), 'start');
            }
        },

        touchMove(e) {
            if (this.chart && e.touches.length > 0) {
                var touch = e.touches[0];
                var handler = this.chart.getZr().handler;
                handler.dispatch('mousemove', {
                    zrX: touch.x,
                    zrY: touch.y
                });
                handler.processGesture(wrapTouch(e), 'change');
            }
        },

        touchEnd(e) {
            if (this.chart) {
                const touch = e.changedTouches ? e.changedTouches[0] : {};
                var handler = this.chart.getZr().handler;
                handler.dispatch('mouseup', {
                    zrX: touch.x,
                    zrY: touch.y
                });
                handler.dispatch('click', {
                    zrX: touch.x,
                    zrY: touch.y
                });
                handler.processGesture(wrapTouch(e), 'end');
            }
        }
    },
    created: function () {}
};
function wrapTouch(event) {
    for (let i = 0; i < event.touches.length; ++i) {
        const touch = event.touches[i];
        touch.offsetX = touch.x;
        touch.offsetY = touch.y;
    }
    return event;
}
</script>
<style>
.ec-canvas {
    width: 100%;
    height: 100%;
}
</style>
