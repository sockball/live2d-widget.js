<template>
    <div class='modelList' :class='showList ? "show" : "hidden"' @click='showList = false'>
        <ul class='contextMenu' :style='menuPosition'>
            <li v-for='(name, index) in alias' :key='`_${index}`'
                class='li'
                @mouseenter='setBackground($event)'
                @mouseleave='resetBackground($event)'
                @click='changeModel(name)'
            >
                <span>{{ name }}</span>
            </li>
            <li v-for='(action, index) in actions' :key='index'
                class='li'
                :class='index === 0 ? "division" : ""'
                @click='event(action.name)'
                @mouseenter='setBackground($event)'
                @mouseleave='resetBackground($event)'
            >
                <span>{{ action.label }}</span>
            </li>
        </ul>
    </div>
</template>

<script>
import { L2Dwidget as Live2DWidget } from 'live2d-widget/src/index'

function on (dom, eventName, callback)
{
    dom[`on${eventName}`] = callback
}

function off (dom, eventName)
{
    dom[`on${eventName}`] = null
}

export default {
    data () {
        const model = {
            // 部分模型崩坏...
            // nij: 'ni-j',
            epsilon: 'epsilon2_1',
            gf: 'gf',
            nico: 'nico',
            chitose: 'chitose',
            haru私服: 'haru01',
            haru制服: 'haru02',
            haruto: 'haruto',
            koharu: 'koharu',
            hibiki: 'hibiki',
            黑猫: 'hijiki',
            izumi: 'izumi',
            初音: 'miku',
            n: 'nietzsche',
            nipsilon: 'nipsilon',
            nito: 'nito',
            shizuku: 'shizuku',
            白猫: 'tororo',
            tsumiki: 'tsumiki',
            大鳥こはく: 'unitychan',
            汪酱: 'wanko',
            z16: 'z16',
        }
        const alias = Object.keys(model)

        return {
            model,
            alias,
            dialog: false,
            actions: [
                { name: 'switchDialog', label: '说点什么' },
                { name: 'drop', label: '退出' },
            ],
            currentModel: '',
            showList: false,
            menuPosition: {},
            canvasId: 'live2DCanvas',
            timeout: 60,
        }
    },
    watch: {
        dialog (newValue) {
            this.actions[0].label = newValue ? '别说话了' : '说点什么'
        }
    },
    methods: {
        event (action) {
            this[action]()
        },
        drop () {
            // 退出
            document.getElementById(this.canvasId).remove()
        },
        switchDialog () {
            // 请求还存在...
            this.dialog = !this.dialog
            this.init(true)
        },
        setBackground (e) {
            e.target.classList.add('enter')
        },
        resetBackground (e) {
            e.target.classList.remove('enter')
        },
        changeModel (alias) {
            this.currentModel = this.getModelSrc(alias)
            this.create(this.currentModel)
            this.eventInit()
        },
        create (model) {
            Live2DWidget.init({
                model: {
                    jsonPath: model,
                },
                display: {
                    // width、height实际值为其值 * 超采样等级(默认2, 即2倍)
                    width: 200,
                    height: 400,
                    position: 'left',
                    hOffset: 0,
                    vOffset: 0,
                },
                mobile: {
                    show: false,
                },
                name: {
                    canvas: this.canvasId,
                },
                react: {
                    opacity: 0.8,
                },
                dev: {
                    border: false,
                },
                // 跨域问题...
                dialog: {
                    enable: this.dialog,
                    hitokoto: true,
                }
            })
        },
        getModelSrc (aliasName) {
            const modelName = this.model[aliasName]
            /* eslint-disable */
            switch (modelName)
            {
                case 'epsilon2_1':
                    return 'https://unpkg.com/live2d-widget-model-epsilon2_1@latest/assets/Epsilon2.1.model.json'

                case 'gf':
                    return 'https://unpkg.com/live2d-widget-model-gf@latest/assets/Gantzert_Felixander.model.json'

                case 'haru01':
                    return 'https://unpkg.com/live2d-widget-model-haru@latest/01/assets/haru01.model.json'

                case 'haru02':
                    return 'https://unpkg.com/live2d-widget-model-haru@latest/02/assets/haru02.model.json'

                case 'nietzsche':
                    // 文件名漏个字母...
                    return 'https://unpkg.com/live2d-widget-model-nietzsche@1.0.5/assets/nietzche.model.json'

                default:
                    return `https://unpkg.com/live2d-widget-model-${modelName}@latest/assets/${modelName}.model.json`
            }
            /* eslint-enable */
        },
        mouseWheel (canvas) {
            // canvas区域内鼠标滚轮调整大小
            on(canvas, 'mousewheel', e => {
                const event = e || window.event
                // 阻止默认滚动条滚动
                event.preventDefault()

                // 以chrome为例 更多 @see https://www.cnblogs.com/caoruiy/p/4694498.html
                // 向下滚动为负值 值为120的倍数 times作为倍率 value为单倍变化的值
                const times = event.wheelDelta / 120
                const value = 4 * times
                const width = Number.parseInt(canvas.style.width || canvas.width)
                const height = Number.parseInt(canvas.style.height || canvas.height)
                canvas.style.width = width + value + 'px'
                canvas.style.height = height + value + 'px'
            })
        },
        drag (canvas) {
            // canvas拖拽
            on(canvas, 'mousedown', e => {
                const event = e || window.event
                // 仅左键可拖动
                if (event.button === 0)
                {
                    // 记录mousedown时鼠标在canvas内的坐标
                    const canvasX = event.clientX - canvas.offsetLeft
                    const canvasY = event.clientY - canvas.offsetTop
                    on(document, 'mousemove', e => {
                        const moveEvent = e || window.event
                        canvas.style.left = moveEvent.clientX - canvasX + 'px'
                        canvas.style.top = moveEvent.clientY - canvasY + 'px'
                    })
                }
            })
            // 拖拽结束后解绑move事件
            on(canvas, 'mouseup', () => {
                off(document, 'mousemove')
            })
        },
        contextMenu (canvas) {
            // 右击菜单切换模型等
            on(canvas, 'contextmenu', e => {
                const event = e || window.event
                event.preventDefault()
                const menuHeight = 200
                this.menuPosition = {
                    left: (event.clientX + 10) + 'px',
                    top: (event.clientY - menuHeight) + 'px',
                    display: 'block',
                }
                this.showList = true
            })
        },
        init (reload = false) {
            if (reload === false)
            {
                // 第一次随机加载一个... Math.random得到 [0, 1) 区间的数
                // const num = Math.floor(Math.random() * alias.length)
                const randomAlias = this.alias[6]// 20]
                this.currentModel = this.getModelSrc(randomAlias)
            }
            this.create(this.currentModel)
            this.eventInit()
        },
        eventInit () {
            let time = 0
            const canvasId = this.canvasId
            let interval = setInterval(() => {
                time++
                if (time > this.timeout)
                {
                    clearInterval(interval)
                }
                const canvas = document.getElementById(canvasId)
                if (canvas)
                {
                    clearInterval(interval)
                    // 解除事件限制...
                    canvas.style.setProperty('pointer-events', 'initial')
                    this.mouseWheel(canvas)
                    this.drag(canvas)
                    this.contextMenu(canvas)
                }
            }, 1000)
        }
    },
    created () {
        this.init()
    }
}
</script>

<style scoped>
    .modelList {
        position: fixed;
        left: 0;
        top: 0;
        width: 100%;
        height: 100%;
        z-index: 9999;
    }
    .contextMenu {
        position: absolute;
        border-radius: 4px;
        left: 300px;
        color: #F5D399;
        width: 150px;
        height: 200px;
        overflow: auto;
        background: rgba(0, 173, 181, .8);
        /* background: #00adb5; */
        box-shadow: 0 3px 12px rgba(0, 173, 181, .8);
    }
    .li {
        list-style: none;
        padding: 3px;
        padding-left: 20px;
        cursor: pointer;
    }
    .division {
        border-top: 1px solid rgba(255,255,255,0.5);
        padding-top: 6px;
    }
    .enter {
        background: rgb(204, 204, 255, .7)
    }
    .show {
        display: block;
    }
    .hidden {
        display: none;
    }
    /* 滚动条样式 仅chrome适用... 参考 https://qishaoxuan.github.io/css_tricks/scrollTemp/ */
    .contextMenu::-webkit-scrollbar {
        width: 8px;
        height: 50px;
        background: rgb(204, 204, 255, .3);
    }
     .contextMenu::-webkit-scrollbar-corner,
     .contextMenu::-webkit-scrollbar-thumb,
     .contextMenu::-webkit-scrollbar-track {
       border-radius: 4px;
     }
     /* 滚动条轨道 */
     .contextMenu::-webkit-scrollbar-corner,
     .contextMenu::-webkit-scrollbar-track {
       background-color: rgba(180, 160, 120, 0.1);
       box-shadow: inset 0 0 1px rgba(180, 160, 120, 0.5);
     }
     /* 滚动条手柄 */
     .contextMenu::-webkit-scrollbar-thumb {
       background-color: pink;
     }
</style>
