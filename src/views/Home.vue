<template>
    <my-page title="CSS3 颜色渐变选择器" :page="page">
        <div class="gradient-box">
            <div class="preview" :style="previewStyle"></div>
            <div class="slider">
                <div class="thumb" :style="style(stop)" v-for="stop, index in stops" @mousedown="mouseDown($event, stop, index)"></div>
            </div>
            <div class="btns">
                <ui-icon-button class="btn" icon="add" @click="add" title="添加停止点" />
                <ui-select-field class="input" v-model="type" label="类型">
                    <ui-menu-item value="linear" title="线性渐变"/>
                    <ui-menu-item value="circle" title="径向渐变 - 圆形"/>
                    <ui-menu-item value="ellipse" title="径向渐变 - 椭圆"/>
                </ui-select-field>
                <ui-select-field class="input" v-model="direction" label="方向" v-if="type === 'linear'">
                    <ui-menu-item value="left" title="左"/>
                    <ui-menu-item value="right" title="右"/>
                    <ui-menu-item value="bottom" title="顶部"/>
                    <ui-menu-item value="top" title="底部"/>
                </ui-select-field>
                <ui-select-field class="input" v-model="hCenter" label="水平中心" v-if="type !== 'linear'">
                    <ui-menu-item value="center" title="居中"/>
                    <ui-menu-item value="left" title="左"/>
                    <ui-menu-item value="right" title="右"/>
                </ui-select-field>
                <ui-select-field class="input" v-model="vCenter" label="垂直中心" v-if="type !== 'linear'">
                    <ui-menu-item value="center" title="居中"/>
                    <ui-menu-item value="top" title="顶部"/>
                    <ui-menu-item value="bottom" title="底部"/>
                </ui-select-field>
            </div>
        </div>
        <ui-color-picker v-model="stop.color" v-if="stop" ref="colorPicker" />
        <ui-raised-button label="删除停止点" @click="removeStop" v-if="stops.length > 2" />
        <!-- 偏移量 -->
        <!-- 渐变关键颜色结点 -->
        <!-- {{ stop.color }} -->
        <div class="preview-title">预览</div>
        <div class="preview-box">
            <div class="preview-1" :style="previewStyle"></div>
            <div class="preview-2" :style="previewStyle"></div>
        </div>
        <ui-drawer class="code-box" right :open="codeVisible" @close="toggleCode()">
            <ui-appbar title="代码">
                <ui-icon-button icon="close" slot="left" @click="toggleCode" />
            </ui-appbar>
            <div class="body">
                <pre><code>{{ code2 }}</code></pre>
            </div>
        </ui-drawer>
    </my-page>
</template>

<script>
    export default {
        data () {
            return {
                type: 'linear',
                direction: 'right',
                hCenter: 'center',
                vCenter: 'center',
                stopIndex: 0,
                stops: [
                    {
                        color: '#d5f437',
                        position: 0.36
                    },
                    {
                        color: '#7a64f6',
                        position: 0.75
                    }
                ],
                stop: {
                    color: '#f00'
                },
                code2: '',
                // stop: this.stops[0],
                codeVisible: false,
                page: {
                    menu: [
                        {
                            type: 'icon',
                            icon: 'code',
                            click: this.toggleCode,
                            title: '显示代码'
                        }
                    ]
                }
            }
        },
        computed: {
            previewStyle() {
                return {
                    background: this.getAttributeValue()
                }
            }
        },
        mounted() {
            this.stop = this.stops[0]
            document.addEventListener('mousemove', this._onMouseMove = e => {
                this.mouseMove(e)
            }, false)
            document.addEventListener('mouseup', this._onMouseUp = e => {
                this.mouseUp(e)
            }, false)
        },
        destroyed() {
            document.removeEventListener('mousemove', this._onMouseMove)
            document.removeEventListener('mousemove', this._onMouseUp)
        },
        methods: {
            removeStop() {
                this.stops.splice(this.stopIndex, 1)
            },
            getAttributeValue() {
                let stops = this.stops.sort((a, b) => {
                    return a.position - b.position
                })
                let code = ''
                for (let stop of stops) {
                    code += ', ' + stop.color + ' ' + (stop.position.toFixed(2) * 100) + '%'
                }
                let bg
                if (this.type === 'linear') {
                    bg = `linear-gradient(to ${this.direction}${code})`
                } else if (this.type === 'circle') {
                    bg = `radial-gradient(circle at ${this.hCenter} ${this.vCenter}${code})`
                } else {
                    bg = `radial-gradient(ellipse at ${this.hCenter} ${this.vCenter}${code})`
                }
                console.log(bg)
                return bg
            },
            toggleCode() {
                this.codeVisible = !this.codeVisible
                if (this.codeVisible) {
                    let bg = this.getAttributeValue()
                    this.code2 = `background-image: ${bg}`
                }
            },
            add() {
                this.stops.push({
                    color: '#000',
                    position: 1
                })
            },
            mouseMove(e) {
                if (this.isMouseDown) {
                    let position = this.originPosition + (e.pageX - this.mouseDownX) / 402
                    if (position < 0) {
                        position = 0
                    }
                    if (position > 1) {
                        position = 1
                    }
                    this.stop.position = position
                }
            },
            mouseUp(e) {
                this.isMouseDown = false
            },
            mouseDown(e, stop, index) {
                // stop.position = 1
                this.stop = stop
                this.stopIndex = index
                this.isMouseDown = true
                this.mouseDownX = e.pageX
                this.mouseDownY = e.pageY
                this.originPosition = stop.position
                console.log(this.$refs.colorPicker)
            },
            style(stop, index) {
                return {
                    left: (402 * stop.position - 6) + 'px',
                    'background-color': stop.color
                }
            }
        }
    }
</script>

<style lang="scss" scoped>
.gradient-box {
    // width: 472px;
    // height: 204px;
    padding: 16px;
    // background: linear-gradient(top, #FAFAFA 0%, #DFDFDF 100%);
    box-shadow: 0 1px 6px rgba(0,0,0,.117647), 0 1px 4px rgba(0,0,0,.117647);
    user-select: none;
    .preview {
        width: 402px;
        height: 44px;
        // background: -webkit-linear-gradient(left, #d5f437 36%, #7a64f6 75%);
    }
    .slider {
        position: relative;
        width: 402px;
        height: 2px;
        margin-top: 8px;
        margin-bottom: 16px;
        background-color: #bdbdbd;
        // background-color: #f00;
    }
    .thumb {
        position: absolute;
        top: -4px;
        left: 10px;
        width: 12px;
        height: 12px;
        border-radius: 50%;
        background-color: #009688;
        border: 1px solid #000;
        cursor: move;
    }
    .btns {
        // margin-bottom: 16px;
        .btn {
            margin-right: 8px;
        }
    }
}
.input {
    width: 160px;
    margin-right: 8px;
}
.preview-title {
    margin: 16px 0;
    color: #999;
}
.preview-1 {
    float: left;
    width: 160px;
    height: 160px;
    margin-right: 16px;
    border: 1px solid #000;
    border-radius: 50%;
}
.preview-2 {
    float: left;
    width: 240px;
    height: 160px;
    border: 1px solid #000;
}
.code-box {
    width: 100%;
    max-width: 400px;
    .body {
        padding: 16px;
    }
    pre {
        // white-space: nowrap;
        overflow: auto;
    }
}
</style>
