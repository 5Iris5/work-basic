<!--
 * @Author: Larissa
 * @Date: 2021-12-14 16:24:52
 * @LastEditTime: 2021-12-31 16:54:21
 * @LastEditors: Please set LastEditors
 * @Description: 打开koroFileHeader查看配置 进行设置: https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE
 * @FilePath: \oa_web\src\components\dialog\customColumn.vue
-->
<template>
    <div class="custom-column">
        <el-dialog
            v-moveallow
            :close-on-click-modal="false"
            title="自定义列"
            custom-class="customColumnDialog"
            :visible.sync="customColumnVisible"
            :before-close="closeCustomColumn"
        >
            <div class="content">
                <div class="content_left">
                    <div
                        v-for="(item, index) in columnListTmp"
                        :key="index"
                        :class="['content_l_side', curretIndex == index ? 'content_l_side_active' : '']"
                        @click="toCurrentIndex(index)"
                    >
                        {{ item.label }}
                    </div>
                </div>
                <div class="content_right">
                    <div
                        class="typeList"
                        v-for="(item, index) in columnListTmp"
                        :key="index"
                        :id="'rightTitle' + index"
                    >
                        <el-checkbox
                            :indeterminate="item.isIndeterminate"
                            v-model="item.isCustomize"
                            @change="handleCheckAllChange(item, index, $event)"
                            class="check_title"
                            >{{ item.label }}</el-checkbox
                        >
                        <div class="check_list">
                            <el-popover
                                placement="top-start"
                                popper-class="dialogColumn_popper"
                                v-for="(itemList, i) in item.children"
                                :key="i"
                                trigger="hover"
                                :content="itemList.label"
                                :open-delay="200"
                                :disabled="!itemList.label"
                            >
                                <el-checkbox
                                    :label="itemList.id"
                                    v-model="itemList.isCustomize"
                                    @change="handleCheckedColumnChange(item, index, $event)"
                                    style="outline: 0"
                                    slot="reference"
                                    >{{ itemList.label }}</el-checkbox
                                >
                            </el-popover>
                        </div>
                    </div>
                </div>
            </div>
            <div slot="footer" class="dialog-footer">
                <el-button @click="closeCustomColumn" size="mini">取 消</el-button>
                <el-button type="primary" @click="submitCustomColumn" size="mini">确 定</el-button>
            </div>
        </el-dialog>
    </div>
</template>
<script>
    export default {
        name: 'customColumn',
        props: {
            customColumnVisible: {
                type: Boolean,
                default: false,
            },
            originalColumnList: {
                type: Array,
                default: () => [],
            },
        },
        computed: {
            columnListTmp() {
                let _normal = [],
                    _tmp = []
                this.originalColumnList.forEach((item) => {
                    if (item.children.length) {
                        item['isIndeterminate'] = true
                        _tmp.push(item)
                    } else {
                        _normal.push(item)
                    }
                })
                _tmp.unshift({ label: '常规信息', children: _normal, isIndeterminate: true })
                // console.log('_tmp: ', _tmp)
                return _tmp
            },
        },
        data() {
            return {
                curretIndex: 0,
                fieldIds: [],
            }
        },
        methods: {
            handleCheckAllChange(item, index, val) {
                // console.log(item, index, val)
                this.columnListTmp[index].children.forEach((v) => (v.isCustomize = val ? true : false))
                this.$set(this.columnListTmp[index], 'isIndeterminate', false)
            },
            handleCheckedColumnChange(item, index) {
                // console.log(item, index, val)
                let checkedCount = item.children.filter((v) => v.isCustomize).length
                this.$set(this.columnListTmp[index], 'isCustomize', checkedCount === item.children.length)
                this.$set(
                    this.columnListTmp[index],
                    'isIndeterminate',
                    checkedCount > 0 && checkedCount < item.children.length,
                )
            },

            closeCustomColumn() {
                this.$emit('close-customColumn')
            },
            submitCustomColumn() {
                this.fieldIds = []
                this.getFiledIds(this.columnListTmp)
                let fieldIds = this.fieldIds.filter((v) => v)
                // console.log('originalColumnList: ', this.originalColumnList, fieldIds)
                this.$emit('submit-customColumn', fieldIds, this.originalColumnList)
            },
            getFiledIds(list, bool) {
                list.forEach((v) => {
                    if (v.children.length) this.getFiledIds(v.children, true)
                    if (v.isCustomize) this.fieldIds.push(v.id)
                    if (v.isCustomize && bool) this.fieldIds.push(v.parentId)
                })
            },

            toCurrentIndex(index) {
                this.curretIndex = index
                let dom = document.querySelector('#rightTitle' + index)
                let domParent = document.querySelector('.content_right')
                domParent.scrollTo({
                    top: dom.offsetTop - 77,
                    behavior: 'smooth',
                })
            },
        },
    }
</script>
<style lang="less" scoped>
    .custom-column {
        /deep/.customColumnDialog {
            width: 992px;
            border-radius: 2px;
            .el-dialog__header {
                padding: 32px 32px 26px;
                height: 76px;
            }
            .el-dialog__header .el-dialog__title {
                height: 18px;
                font-size: 18px;
                font-weight: 500;
                color: #17233d;
                line-height: 18px;
                text-shadow: 0 1px 0 rgba(0, 0, 0, 0.5);
            }
            .el-dialog__body {
                padding: 0 40px;
                border: none;
                .content {
                    display: flex;
                    height: 516px;
                    border: 1px solid #eaebec;
                    border-radius: 5px;
                    .content_left {
                        width: 152px;
                        border-right: 1px solid #eaebec;
                        overflow: auto;
                        .content_l_side {
                            padding-left: 16px;
                            line-height: 40px;
                            color: #515a6e;
                            cursor: pointer;
                        }
                        .content_l_side_active {
                            color: #2e71ea;
                            background-color: #eaf1ff;
                        }
                    }
                    .content_right {
                        flex: 1;
                        overflow: auto;
                        .typeList {
                            padding: 16px 0 22px 24px;
                            border-bottom: 1px solid #eaebec;
                            .check_title {
                                font-weight: bold;
                                margin-bottom: 10px;
                            }
                            .check_list {
                                display: flex;
                                flex-wrap: wrap;
                                > span {
                                    display: flex;
                                    width: 33.3%;
                                }
                                .el-checkbox {
                                    margin-right: 0;
                                    margin-bottom: 10px;
                                    display: flex;
                                    align-items: center;
                                    .el-checkbox__label {
                                        overflow: hidden;
                                        white-space: nowrap;
                                        text-overflow: ellipsis;
                                    }
                                }
                            }
                        }
                    }
                }
            }
            .el-dialog__footer {
                /deep/.el-button {
                    width: 15%;
                }
            }
        }
    }
</style>