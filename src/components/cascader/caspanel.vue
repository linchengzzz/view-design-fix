<template>
    <span>
        <ul v-if="data && data.length" :class="[prefixCls + '-menu']">
            <li v-for="data in data" :key="data.value" 
            :class="[`${prefixCls}-menu-item`,
                {[`${prefixCls}-menu-item-active`]: tmpItem.value === data.value,
                    [`${prefixCls}-menu-item-disabled`]: data.disabled
                }]"
                @click="handleClickItem(data)"
                @mouseenter="handleHoverItem(data)">
                <div v-if='data.introduction'>
                    <Tooltip ref='tooltipShow' placement="left" trigger="hover" :content="data.introduction" transfer theme="light" max-width="200" style="width:110%;">
                        <template v-slot:content>
                            <p style="color: #2f2f3f;font-weight: bold;"> {{data.label}}</p>
                            <span>{{data.introduction}}</span>
                        </template>
                        <p style="width:100%;z-index:1000;" :class="data.value">{{ data.label }}</p>
                    </Tooltip>
                    <Icon :type="arrowType" :custom="customArrowType" :size="arrowSize" v-if="data.children && data.children.length" />
                    <i v-if="'loading' in data && data.loading" class="ivu-icon ivu-icon-ios-loading ivu-load-loop ivu-cascader-menu-item-loading"></i>
                </div>
                <div v-else>
                    {{ data.label }}
                    <Icon :type="arrowType" :custom="customArrowType" :size="arrowSize" v-if="data.children && data.children.length" />
                    <i v-if="'loading' in data && data.loading" class="ivu-icon ivu-icon-ios-loading ivu-load-loop ivu-cascader-menu-item-loading"></i>
                </div>
            </li>
        </ul>
        <Caspanel v-if="sublist && sublist.length" :prefix-cls="prefixCls" :data="sublist" :disabled="disabled" :trigger="trigger" :change-on-select="changeOnSelect"></Caspanel>
    </span>
</template>
<script>
    import Casitem from './casitem.vue';
     import Icon from '../icon/icon.vue';
    import Emitter from '../../mixins/emitter';
    import { findComponentUpward, findComponentDownward } from '../../utils/assist';

    let key = 1;

    export default {
        name: 'Caspanel',
        mixins: [ Emitter ],
        components: { Icon,Casitem },
        props: {
            data: {
                type: Array,
                default () {
                    return [];
                }
            },
            disabled: Boolean,
            changeOnSelect: Boolean,
            trigger: String,
            prefixCls: String,
            showLoading:false,
        },
       computed: {
        
            // 3.4.0, global setting customArrow 有值时，arrow 赋值空
            arrowType () {
                let type = 'ios-arrow-forward';

                if (this.$IVIEW) {
                    if (this.$IVIEW.cascader.customItemArrow) {
                        type = '';
                    } else if (this.$IVIEW.cascader.itemArrow) {
                        type = this.$IVIEW.cascader.itemArrow;
                    }
                }
                return type;
            },
            // 3.4.0, global setting
            customArrowType () {
                let type = '';

                if (this.$IVIEW) {
                    if (this.$IVIEW.cascader.customItemArrow) {
                        type = this.$IVIEW.cascader.customItemArrow;
                    }
                }
                return type;
            },
            // 3.4.0, global setting
            arrowSize () {
                let size = '';

                if (this.$IVIEW) {
                    if (this.$IVIEW.cascader.itemArrowSize) {
                        size = this.$IVIEW.cascader.itemArrowSize;
                    }
                }
                return size;
            }
        },
        data () {
            return {
                tmpItem: {},
                result: [],
                sublist: []
            };
        },
        watch: {
            data () {
                this.sublist = [];
            }
        },
        methods: {
            handleClickItem (item) {
                if (this.trigger !== 'click' && item.children && item.children.length) return;  // #1922
                this.handleTriggerItem(item, false, true);
            },
            handleHoverItem (item) {
                if (this.trigger !== 'hover' || !item.children || !item.children.length) return;  // #1922
                this.handleTriggerItem(item, false, true);
            },
            handleTriggerItem (item, fromInit = false, fromUser = false) {
                if (item.disabled) return;

                const cascader = findComponentUpward(this, 'Cascader');
                if (item.loading !== undefined && !item.children.length) {
                    if (cascader && cascader.loadData) {
                        cascader.loadData(item, () => {
                            // todo
                            if (fromUser) {
                                cascader.isLoadedChildren = true;
                            }
                            if (item.children.length) {
                                this.handleTriggerItem(item);
                            }
                        });
                        return;
                    }
                }

                // return value back recursion  // 向上递归，设置临时选中值（并非真实选中）
                const backItem = this.getBaseItem(item);
                // #5021 for this.changeOnSelect，加 if 是因为 #4472
                if (
                    this.changeOnSelect ||
                    (backItem.label !== this.tmpItem.label || backItem.value !== this.tmpItem.value) ||
                    (backItem.label === this.tmpItem.label && backItem.value === this.tmpItem.value)
                ) {
                    this.tmpItem = backItem;
                    this.emitUpdate([backItem]);
                }

                if (item.children && item.children.length){
                    this.sublist = item.children;
                    this.dispatch('Cascader', 'on-result-change', {
                        lastValue: false,
                        changeOnSelect: this.changeOnSelect,
                        fromInit: fromInit
                    });

                    // #1553
                    if (this.changeOnSelect) {
                        const Caspanel = findComponentDownward(this, 'Caspanel');
                        if (Caspanel) {
                            Caspanel.$emit('on-clear', true);
                        }
                    }
                } else {
                    this.sublist = [];
                    this.dispatch('Cascader', 'on-result-change', {
                        lastValue: true,
                        changeOnSelect: this.changeOnSelect,
                        fromInit: fromInit
                    });
                }

                if (cascader) {
                    cascader.$refs.drop.update();
                }
            },
            updateResult (item) {
                this.result = [this.tmpItem].concat(item);
                this.emitUpdate(this.result);
            },
            getBaseItem (item) {
                let backItem = Object.assign({}, item);
                if (backItem.children) {
                    delete backItem.children;
                }

                return backItem;
            },
            emitUpdate (result) {
                if (this.$parent.$options.name === 'Caspanel') {
                    this.$parent.updateResult(result);
                } else {
                    this.$parent.$parent.updateResult(result);
                }
            },
            getKey () {
                return key++;
            }
        },
        mounted () {
            this.$on('on-find-selected', (params) => {
                const val = params.value;
                let value = [...val];
                for (let i = 0; i < value.length; i++) {
                    for (let j = 0; j < this.data.length; j++) {
                        if (value[i] === this.data[j].value) {
                            this.handleTriggerItem(this.data[j], true);
                            value.splice(0, 1);
                            this.$nextTick(() => {
                                this.broadcast('Caspanel', 'on-find-selected', {
                                    value: value
                                });
                            });
                            return false;
                        }
                    }
                }
            });
            // deep for #1553
            this.$on('on-clear', (deep = false) => {
                this.sublist = [];
                this.tmpItem = {};
                if (deep) {
                    const Caspanel = findComponentDownward(this, 'Caspanel');
                    if (Caspanel) {
                        Caspanel.$emit('on-clear', true);
                    }
                }
            });
        }
    };
</script>
