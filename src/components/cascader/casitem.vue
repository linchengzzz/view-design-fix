<template>
    <li :class="classes">
        <div v-if='data.introduction'>
            <Tooltip placement="left" trigger="hover" :content="data.introduction" transfer theme="light" max-width="200" style="width:110%;">
                <template v-slot:content>
                    <p style="color: #2f2f3f;font-weight: bold;"> {{data.label}}</p>
                    <span>{{data.introduction}}</span>
                </template>
                <p style="width:100%;">{{ data.label }}</p>
            </Tooltip>
            <Icon :type="arrowType" :custom="customArrowType" :size="arrowSize" v-if="showArrow" />
            <i v-if="showLoading" class="ivu-icon ivu-icon-ios-loading ivu-load-loop ivu-cascader-menu-item-loading"></i>
        </div>
        <div v-else>
            {{ data.label }}
            <Icon :type="arrowType" :custom="customArrowType" :size="arrowSize" v-if="showArrow" />
            <i v-if="showLoading" class="ivu-icon ivu-icon-ios-loading ivu-load-loop ivu-cascader-menu-item-loading"></i>
        </div>
    </li>
</template>
<script>
    import Icon from '../icon/icon.vue';
    import Tooltip from '../tooltip/tooltip.vue';

    export default {
        name: 'Casitem',
        components: { Icon,Tooltip },
        props: {
            data: Object,
            prefixCls: String,
            tmpItem: Object
        },
        computed: {
            classes () {
                return [
                    `${this.prefixCls}-menu-item`,
                    {
                        [`${this.prefixCls}-menu-item-active`]: this.tmpItem.value === this.data.value,
                        [`${this.prefixCls}-menu-item-disabled`]: this.data.disabled
                    }
                ];
            },
            showArrow () {
                return (this.data.children && this.data.children.length) || ('loading' in this.data && !this.data.loading);
            },
            showLoading () {
                return 'loading' in this.data && this.data.loading;
            },
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
        }
    };
</script>
