<!--
  Tencent is pleased to support the open source community by making 蓝鲸智云PaaS平台社区版 (BlueKing PaaS Community Edition) available.
  Copyright (C) 2017-2019 THL A29 Limited, a Tencent company. All rights reserved.
  Licensed under the MIT License (the "License"); you may not use this file except in compliance with the License.
  You may obtain a copy of the License at
  http://opensource.org/licenses/MIT
  Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on
  an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the
  specific language governing permissions and limitations under the License.
-->

<template>
    <main class="app-main">
        <div class="main-top">
            <div class="page-title">
                <div class="page-name">
                    可视化布局
                </div>
                <!-- <div class="back">
                    <i class="bk-drag-icon bk-drag-arrow-back"></i>
                </div>
                <div class="page-name">
                    单据详情页【配置平台】
                </div> -->
            </div>
            <div class="function-and-tool">
                <ul class="function-tabs">
                    <li class="tab-item" @click="handleToolAction('edit')" :class="actionSelected === 'edit' ? 'active' : ''">编辑</li>
                    <li class="tab-item" @click="handleToolAction('vueCode')" :class="actionSelected === 'vueCode' ? 'active' : ''">Vue 源码</li>
                </ul>
                <div class="tool-actions">
                    <!-- <div class="action-item" v-bk-tooltips="{ content: '撤销', placements: ['bottom'] }">
                        <i class="bk-drag-icon bk-drag-undo"></i>
                    </div> -->
                    <div class="action-item" :class="actionSelected === 'preview' ? 'active' : ''"
                        v-bk-tooltips="{ content: '预览', placements: ['bottom'] }"
                        @click="handlePreview">
                        <i class="bk-drag-icon bk-drag-play"></i>
                    </div>
                    <div class="action-item" :class="actionSelected === 'del' ? 'active' : ''"
                        v-bk-tooltips="{ content: '清空', placements: ['bottom'] }"
                        @click="handleClearAll">
                        <i class="bk-drag-icon bk-drag-delete"></i>
                    </div>
                    <!-- <div class="action-item" @click="test">
                        test
                    </div>
                    <div class="action-item" @click="test1">
                        mock
                    </div> -->
                    <div class="action-item text" @click="showFunManage">
                        函数管理
                    </div>
                    <!-- <div class="action-item">
                        <i class="bk-drag-icon bk-drag-play"></i>
                    </div>
                    <div class="action-item">
                        <i class="bk-drag-icon bk-drag-play"></i>
                    </div>
                    <div class="action-item">
                        <i class="bk-drag-icon bk-drag-play"></i>
                    </div> -->
                </div>
            </div>
        </div>
        <div class="main-container">
            <aside class="main-left-sidebar" :class="{ 'is-collapse': collapseSide.left }">
                <div class="sidebar-panel">
                    <div class="sidebar-hd">
                        <ul class="category-tabs">
                            <li class="tab-item active">组件</li>
                            <!-- <li class="tab-item">图标</li> -->
                            <!-- <li class="tab-item">层级结构</li> -->
                        </ul>
                        <div class="search-bar">
                            <component-search :result.sync="componentSearchResult" />
                        </div>
                    </div>
                    <div class="sidebar-bd">
                        <template v-for="(group, groupIndex) in componentGroupList">
                            <div
                                v-show="isShowComponentGroup(group)"
                                :key="groupIndex"
                                :class="getComponentGroupClass(group)">
                                <div class="group-title" @click="componentGroupFolded[group] = !componentGroupFolded[group]">
                                    <i class="bk-drag-icon bk-drag-arrow-down"></i>
                                    {{group}}
                                </div>
                                <div class="group-content">
                                    <vue-draggable
                                        class="source-drag-area component-list"
                                        :list="componentGroups[group]"
                                        :sort="false"
                                        :group="draggableSourceGroup"
                                        :force-fallback="false"
                                        ghost-class="source-ghost"
                                        chosen-class="source-chosen"
                                        drag-class="source-drag"
                                        :clone="cloneFunc"
                                        @start="sourceAreaStartHandler"
                                        @choose="onChoose($event, group)"
                                        @end="sourceAreaEndHandler"
                                    >
                                        <template v-for="(component, componentIndex) in componentGroups[group]">
                                            <!-- @mouseenter="handleComponentMouseenter($event, component)" -->
                                            <div class="component-item" :class="placeholderElemDisplay" :key="componentIndex"
                                                v-show="!componentSearchResult || component.displayName === componentSearchResult.displayName">
                                                <div class="component-icon">
                                                    <i class="bk-drag-icon" :class="component.icon"></i>
                                                </div>
                                                <div class="component-name" v-if="component.displayName">{{component.displayName}}</div>
                                            </div>
                                        </template>
                                    </vue-draggable>
                                </div>
                            </div>
                        </template>
                    </div>
                </div>
                <i
                    class="bk-drag-icon bk-drag-angle-left collapse-icon"
                    v-bk-tooltips.right="{ content: '查看所有组件', disabled: !collapseSide.left }"
                    @click="handleCollapseSide('left')">
                </i>
            </aside>

            <!-- 这里用 v-show，切换源码或者预览时，如果时 v-if，那么 grid.vue 里的 renderDataSlot 会重置，这个值并没有存在 store 中 -->
            <div class="main-content" :class="mainContentClass" @click="dragWrapperClickHandler" v-show="actionSelected === 'edit'">
                <vue-draggable
                    :key="refreshDragAreaKey"
                    class="target-drag-area"
                    :list="targetData"
                    :sort="true"
                    :group="draggableTargetGroup"
                    ghost-class="target-ghost"
                    chosen-class="target-chosen"
                    drag-class="target-drag"
                    @change="log"
                    @end="targetAreaEndHandler"
                >
                    <render-grid v-for="item in targetData" :key="item.componentId" :component-data="item">
                    </render-grid>
                </vue-draggable>
            </div>
            <!-- <div class="main-content" v-if="actionSelected === 'preview'">
                <preview class="preview-area" :target-data="targetData"></preview>
            </div> -->
            <div class="main-content" :class="mainContentClass" v-if="actionSelected === 'vueCode'">
                <vue-code class="code-area" :target-data="targetData"></vue-code>
            </div>

            <aside class="main-right-sidebar" :class="{ 'is-collapse': collapseSide.right }">
                <div class="selected-component-info" v-if="curSelectedComponentData.componentId && !collapseSide.right">
                    <div class="component-id">{{curSelectedComponentData.componentId}}</div>
                    <div class="action-wrapper">
                        <bk-button title="primary" @click="showDeleteElement" id="del-component-right-sidebar">删除</bk-button>
                    </div>
                </div>
                <material-modifier />
                <i
                    class="bk-drag-icon bk-drag-angle-left collapse-icon"
                    v-bk-tooltips.right="{ content: '查看组件配置', disabled: !collapseSide.right }"
                    @click="handleCollapseSide('right')">
                </i>
            </aside>
        </div>

        <Methods :is-show.sync="isShowFun"></Methods>

        <bk-dialog v-model="delComponentConf.visiable"
            class="del-component-dialog"
            theme="primary"
            :mask-close="false"
            :header-position="delComponentConf.headerPosition"
            title="删除"
            @confirm="confirmDelComponent"
            @cancel="cancelDelComponent"
            @after-leave="afterLeaveDelComponent">
            <div>
                <p>确认删除{{delComponentConf.item.name}}组件【{{delComponentConf.item.componentId}}】？</p>
                <p class="del-grid-tip" v-if="delComponentConf.item.type === 'render-grid'">删除grid元素会连带删除grid中所有子元素</p>
            </div>
        </bk-dialog>
    </main>
</template>

<script>
    import { mapGetters, mapMutations } from 'vuex'
    import cloneDeep from 'lodash.clonedeep'

    import componentList from '@/element-materials/materials'
    import { uuid, removeClassWithNodeClass } from '@/common/util'
    import RenderGrid from '@/components/render/grid'
    import MaterialModifier from '@/element-materials/modifier'
    // import Preview from '@/components/preview'
    import VueCode from '@/components/vue-code'
    import Methods from '@/components/methods'
    import codeMixin from '@/components/vue-code/code-mixin'
    import ComponentSearch from './component-search'

    import customComponents from '@/custom'

    // import Img from '@/images/component-preview.png'
    export default {
        components: {
            RenderGrid,
            MaterialModifier,
            // Preview,
            VueCode,
            Methods,
            ComponentSearch,

            ...customComponents
        },
        mixins: [codeMixin],
        data () {
            const componentGroupFolded = {}
            const componentGroupList = ['栅格布局', '基础', '表单', '导航', '数据', '反馈', '自定义组件']
            componentGroupList.forEach(group => {
                componentGroupFolded[group] = false
            })

            return {
                componentList: componentList,
                componentGroupList,
                collapseSide: {
                    left: false,
                    right: false
                },
                componentGroupFolded,
                actionSelected: 'edit',
                curDragingComponent: null,
                isShowFun: false,
                componentSearchResult: null,
                refreshDragAreaKey: +new Date(),
                delComponentConf: {
                    visiable: false,
                    headerPosition: 'left',
                    item: {}
                },
                placeholderElemDisplay: ''
            }
        },
        computed: {
            ...mapGetters(['mainContentLoading']),
            ...mapGetters('drag', [
                'draggableSourceGroup',
                'draggableTargetGroup',
                'curSelectedComponentData',
                'functionGroup',
                'pageData'
            ]),

            mainContentClass () {
                return {
                    'collapse-none': !this.collapseSide.left && !this.collapseSide.right,
                    'collapse-one-side': (this.collapseSide.left && !this.collapseSide.right) || (!this.collapseSide.left && this.collapseSide.right),
                    'collapse-both-side': this.collapseSide.left && this.collapseSide.right
                }
            },

            targetData: {
                get () {
                    return this.$store.state.drag.targetData
                },
                set (value) {
                    this.setTargetData(value)
                }
            },

            componentGroups () {
                const componentGroups = {}

                // 分组
                componentList.forEach(component => {
                    const groupName = component.group
                    const componentGroup = componentGroups[groupName]
                    if (componentGroup) {
                        componentGroup.push(component)
                    } else {
                        componentGroups[groupName] = [component]
                    }
                })

                // 组内排序
                Object.values(componentGroups).forEach(list => {
                    list.sort((a, b) => a.order - b.order)
                })

                return componentGroups
            }
        },
        created () {
            this.setFunctionGroup(this.functionGroup)

            const mockCurSelectComponentData = {
                componentId: 'grid-' + uuid(),
                name: 'grid',
                type: 'render-grid',
                renderProps: {
                    'margin-horizontal': {
                        type: 'number',
                        val: 0
                    },
                    'margin-vertical': {
                        type: 'number',
                        val: 0
                    },
                    slots: {
                        type: 'column',
                        val: [{ span: 1, children: [], width: '100%' }]
                    }
                },
                renderStyles: {},
                renderEvents: {}
            }

            this.curDragingComponent = Object.assign({}, mockCurSelectComponentData)
            this.setCurSelectedComponentData(this.curDragingComponent)
            this.setTargetData([this.curDragingComponent])

            window.test = this.test
            window.test1 = this.test1
        },
        mounted () {
            window.addEventListener('beforeunload', function (e) {
                const confirmationMessage = '...';
                (e || window.event).returnValue = confirmationMessage
                return confirmationMessage
            })
        },
        methods: {
            ...mapMutations('drag', [
                'setTargetData',
                'setDraggableSourceGroup',
                'setCurSelectedComponentData',
                'setFunctionGroup'
            ]),

            /***
             * 显示函数管理面版
             */
            showFunManage () {
                this.isShowFun = true
            },

            /**
             * 收起/显示左侧、右侧边栏
             *
             * @param {string} side 左、右方向标识
             */
            handleCollapseSide (side) {
                this.collapseSide[side] = !this.collapseSide[side]
            },

            /**
             * 左侧组件列表 hover 事件
             *
             * @param {Object} e 事件对象
             * @param {Object} component 组件对象
             */
            handleComponentMouseenter (e, component) {
                let popoverInstance = e.target._tippy
                // const componentPreviewImg = `component-preview/${component.name}.png`
                const componentPreviewImg = 'component-preview.png'
                if (!popoverInstance) {
                    popoverInstance = this.$bkPopover(e.target, {
                        content: `<img width="210" src="${require(`@/images/${componentPreviewImg}`)}" />`,
                        boundary: 'viewport',
                        placement: 'right-start',
                        animation: 'fade',
                        duration: [500, 50],
                        theme: 'light popover-component',
                        onShow (instance) {
                            const left = 300 - instance.reference.offsetLeft - 60
                            instance.set({
                                distance: left
                            })
                        },
                        delay: 0
                    })
                }
                popoverInstance.show()
            },

            /**
             * drag-wrapper 拖拽容器点击事件回调，用于把当前点击的组件设置为空
             *
             * @param {Object} e 事件对象
             */
            dragWrapperClickHandler (e) {
                removeClassWithNodeClass('.bk-layout-grid-row', 'selected')
                removeClassWithNodeClass('.component-wrapper', 'selected')

                this.setCurSelectedComponentData({})
            },

            /**
             * 左侧组件列表区域拖拽 choose 回调函数
             * 事件触发顺序 onChoose cloneFunc onStart moveFunc(n) onEnd
             *
             * @param {Object} e 事件对象
             * @param {string} group group 标识
             */
            onChoose (e, group) {
                const uid = uuid()
                const component = cloneDeep(this.componentGroups[group][e.oldIndex])
                const { name = '', type = '', props = {} } = component
                const id = component.name + '-' + uid

                const renderProps = {}
                Object.keys(props).forEach(k => {
                    if (props[k].hasOwnProperty('val') && props[k].val !== '') {
                        renderProps[k] = props[k]
                    }
                })

                const defaultStyles = {}
                const styles = component.styles || []
                const componentDefaultStyles = component.defaultStyles || {}
                styles.forEach(st => {
                    if (st === 'size') {
                        if (componentDefaultStyles.hasOwnProperty('height')) {
                            defaultStyles['height'] = component.defaultStyles['height']
                        }
                        if (componentDefaultStyles.hasOwnProperty('width')) {
                            defaultStyles['width'] = component.defaultStyles['width']
                        }
                    } else {
                        if (componentDefaultStyles.hasOwnProperty(st)) {
                            defaultStyles[st] = component.defaultStyles[st]
                        }
                    }
                })
                if (defaultStyles.hasOwnProperty('display')) {
                    this.placeholderElemDisplay = defaultStyles['display']
                } else {
                    this.placeholderElemDisplay = 'block'
                }

                this.curDragingComponent = {
                    componentId: id,
                    name,
                    type,
                    renderProps: renderProps,
                    renderStyles: { ...defaultStyles },
                    renderEvents: {}
                }
            },

            /**
             * vue-draggable clone 回调函数
             *
             * @param {Object} original 当前拖拽的对象（左侧组件列表中的组件）
             */
            cloneFunc (original) {
                this.setDraggableSourceGroup(Object.assign({}, this.draggableSourceGroup, {
                    name: original.type === 'render-grid' ? 'render-grid' : 'component'
                }))
                console.log('from clone', this.curDragingComponent)
                // debugger
                // // 这里需要换 id，否则同样的组件拖到右边后，id 是一样的，右边同样的组件之间就没法拖动
                // // return Object.assign({}, original, { componentId: uuid() })
                return this.curDragingComponent
            },

            sourceAreaStartHandler (e) {
                console.log('sourceAreaStartHandler', e)
                console.log(this.curDragingComponent)
            },

            /**
             * 左侧组件列表区域拖拽 end 回调函数
             * 当把左侧组件列表区的组件拖入到右侧拖拽区中时会触发（拖拽到右侧拖拽区以及拖拽到右侧拖拽区的组件中两种情况均会触发）
             *
             * @param {Object} e 事件对象
             */
            sourceAreaEndHandler (e) {
                this.placeholderElemDisplay = ''
                console.warn('sourceAreaEndHandler', this.curDragingComponent)
                console.warn('left to right end, targetData: ', this.targetData)
            },

            /**
             * 判断是否显示组件组
             *
             * @param {String} group 组名
             */
            isShowComponentGroup (group) {
                return !this.componentSearchResult || group === this.componentSearchResult.group
            },

            /**
             * 获取组件组的class
             *
             * @param {String} group 组名
             */
            getComponentGroupClass (group) {
                return [
                    'component-group',
                    {
                        'folded': this.componentGroupFolded[group],
                        'search-show': this.componentSearchResult && group === this.componentSearchResult.group
                    }
                ]
            },

            log (evt) {
                // console.warn('evt', evt)
                // const data = evt.moved
                // if (data) {
                //     const targetData = []
                //     targetData.splice(0, 0, ...this.targetData)
                //     // targetData.splice(data.newIndex, 1 , targetData[data.oldIndex])
                //     // 将 data.newIndex 位置上的元素替换为 data.oldIndex 位置的元素，同时返回 [targetData[data.newIndex]]
                //     // 返回的是数组，所以在代码中加入了扩展运算符 ... 将数组转为参数序列。
                //     // 再利用同样的方式将 data.oldIndex 位置上的元素替换为被删除的原数组的 targetData[data.newIndex] 的值。完成交换。
                //     targetData.splice(data.oldIndex, 1, ...targetData.splice(data.newIndex, 1, targetData[data.oldIndex]))
                //     this.targetData.splice(0, this.targetData.length, ...targetData)
                // }
            },

            targetAreaEndHandler (e) {
                // console.error('onEnd11111', e)
            },

            handleToolAction (v) {
                // 点击源码的时候，需要让右侧属性面板消失
                // 如果停留在源码页面时属性面板不消失，这个时候修改属性会生效，预览的时候就会生效，但是源码并不会随着属性的变化而重新生成
                if (v === 'vueCode') {
                    this.dragWrapperClickHandler()
                }
                this.actionSelected = v
            },

            /**
             * 清空页面已拖拽的元素
             */
            handleClearAll () {
                const me = this
                me.$bkInfo({
                    title: '确定清空所有组件元素？',
                    confirmFn () {
                        const mockCurSelectComponentData = {
                            componentId: 'grid-' + uuid(),
                            'name': 'grid',
                            'type': 'render-grid',
                            'renderProps': {
                                'margin-horizontal': {
                                    'type': 'number',
                                    'val': 0
                                },
                                'margin-vertical': {
                                    'type': 'number',
                                    'val': 0
                                },
                                'slots': {
                                    'type': 'column',
                                    'val': [{ 'span': 1, 'children': [], 'width': '100%' }]
                                }
                            },
                            'renderStyles': {},
                            'renderEvents': {}
                        }

                        me.curDragingComponent = Object.assign({}, mockCurSelectComponentData)
                        me.setCurSelectedComponentData(me.curDragingComponent)
                        me.setTargetData([me.curDragingComponent])
                    }
                })
            },

            /**
             * 显示删除选中的元素弹框
             */
            showDeleteElement () {
                this.delComponentConf.visiable = true
                this.delComponentConf.item = Object.assign({}, this.curSelectedComponentData)
            },

            /**
             * 确认删除选中的元素
             */
            confirmDelComponent () {
                const targetData = cloneDeep(this.targetData)
                const { componentId } = this.delComponentConf.item

                const del = (target, cid) => {
                    const len = target.length
                    for (let i = 0; i < len; i++) {
                        const item = target[i]
                        if (item.componentId === cid) {
                            target.splice(i, 1)
                            return target
                        }
                        if (item.children) {
                            del(item.children, cid)
                        } else if (
                            item.renderProps.slots && item.renderProps.slots.type === 'column'
                        ) {
                            del(item.renderProps.slots.val, cid)
                        }
                    }
                    return ''
                }
                del(targetData, componentId)
                this.setTargetData(targetData)
                this.refreshDragAreaKey = +new Date()
                this.setCurSelectedComponentData({})
            },

            /**
             * 取消删除选中的元素弹框
             */
            cancelDelComponent () {
                this.delComponentConf.visiable = false
            },

            /**
             * 删除选中元素弹框 afterLeave 回调
             */
            afterLeaveDelComponent () {
                this.delComponentConf.item = Object.assign({}, {})
            },

            async handlePreview () {
                let errTips = ''
                try {
                    localStorage.removeItem('layout-target-data')
                    localStorage.setItem('layout-target-data', JSON.stringify(this.targetData))
                    const routerUrl = this.$router.resolve({
                        name: 'preview',
                        query: {
                            tmpFile: 'vue-layout-demo' + Math.random().toString().substr(5) + '.vue'
                        }
                    })
                    window.open(routerUrl.href, '_blank')

                    // const code = this.getCode()
                    // const res = await this.$store.dispatch('vueCode/saveVueFile', { code })
                    // console.error(res)
                    // if (res) {
                    //     setTimeout(() => {
                    //         const routerUrl = this.$router.resolve({
                    //             name: 'preview',
                    //             query: {
                    //                 tmpFile: res
                    //             }
                    //         })
                    //         console.error(routerUrl)
                    //         window.open(routerUrl.href, '_blank')
                    //     }, 10)
                    // } else {
                    //     errTips = '预览异常'
                    // }
                } catch (err) {
                    errTips = err.message || err || '预览异常'
                }
                errTips && this.$bkMessage({
                    theme: 'error',
                    message: errTips
                })
            },

            test () {
                console.warn(JSON.stringify(this.targetData))
                console.warn(this.targetData)
                console.warn('pageData', this.pageData)
            },
            test1 (data) {
                if (data) {
                    this.setTargetData(data)
                } else {
                    this.setTargetData([{ 'componentId': 'grid8B6FFD74', 'name': 'grid', 'type': 'render-grid', 'renderProps': { 'margin-horizontal': { 'type': 'number', 'val': 0 }, 'margin-vertical': { 'type': 'number', 'val': 0 }, 'slots': { 'type': 'column', 'val': [{ 'span': 1, 'children': [{ 'componentId': 'input8B753E61', 'name': 'input', 'type': 'bk-input', 'renderProps': { 'value': { 'type': 'string', 'val': 'hello world' }, 'type': { 'type': 'string', 'options': ['text', 'textarea', 'password', 'number', 'email', 'url', 'date'], 'val': 'text' }, 'font-size': { 'type': 'string', 'options': ['normal', 'medium', 'large'], 'val': 'normal' }, 'disabled': { 'type': 'boolean', 'val': false }, 'readonly': { 'type': 'boolean', 'val': false }, 'clearable': { 'type': 'boolean', 'val': true }, 'show-controls': { 'type': 'boolean', 'val': true } }, 'renderStyles': {}, 'renderEvents': {} }, { 'componentId': 'input482301F3', 'name': 'input', 'type': 'bk-input', 'renderProps': { 'value': { 'type': 'string', 'val': 'hello world' }, 'type': { 'type': 'string', 'options': ['text', 'textarea', 'password', 'number', 'email', 'url', 'date'], 'val': 'text' }, 'font-size': { 'type': 'string', 'options': ['normal', 'medium', 'large'], 'val': 'normal' }, 'disabled': { 'type': 'boolean', 'val': false }, 'readonly': { 'type': 'boolean', 'val': false }, 'clearable': { 'type': 'boolean', 'val': true }, 'show-controls': { 'type': 'boolean', 'val': true } }, 'renderStyles': {}, 'renderEvents': {} }, { 'componentId': 'inputB0159B6D', 'name': 'input', 'type': 'bk-input', 'renderProps': { 'value': { 'type': 'string', 'val': 'hello world' }, 'type': { 'type': 'string', 'options': ['text', 'textarea', 'password', 'number', 'email', 'url', 'date'], 'val': 'text' }, 'font-size': { 'type': 'string', 'options': ['normal', 'medium', 'large'], 'val': 'normal' }, 'disabled': { 'type': 'boolean', 'val': false }, 'readonly': { 'type': 'boolean', 'val': false }, 'clearable': { 'type': 'boolean', 'val': true }, 'show-controls': { 'type': 'boolean', 'val': true } }, 'renderStyles': {}, 'renderEvents': {} }], 'width': '50%' }, { 'span': 1, 'children': [{ 'componentId': 'transfer01163C24', 'name': 'transfer', 'type': 'bk-transfer', 'renderProps': { 'display-key': { 'type': 'string', 'val': 'name' }, 'setting-key': { 'type': 'string', 'val': 'id' }, 'sortable': { 'type': 'boolean', 'val': false } }, 'renderStyles': {}, 'renderEvents': {} }], 'width': '50%' }] } }, 'renderStyles': {}, 'renderEvents': {} }])
                }
            }
        }
    }
</script>

<style lang="postcss">
    @import './index.css';
</style>
