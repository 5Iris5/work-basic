<!--
 * @Author: your name
 * @Date: 2021-12-19 09:39:27
 * @LastEditTime: 2021-12-19 15:51:06
 * @LastEditors: Please set LastEditors
 * @Description: 打开koroFileHeader查看配置 进行设置: https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE
 * @FilePath: \oa_web\src\components\view\customSort.vue
-->
<template>
    <div class="cunstom-sort">
        <el-dialog
            v-moveallow
            :close-on-click-modal="false"
            title="排序"
            :visible.sync="sortVisible"
            width="450px"
            :before-close="cancelMethod"
        >
            <el-button size="mini" :disabled="!chooseRowIds.length" @click="insertRow('head')">插入行</el-button>
            <el-button size="mini" :disabled="!chooseRowIds.length" @click="insertRow('behind')">增加行</el-button>
            <el-button
                size="mini"
                type="danger"
                :disabled="!chooseRowIds.length || sortData.length === 1"
                @click="deleteRow"
                >删除</el-button
            >
            <el-table
                stripe
                border
                :data="sortData"
                header-cell-class-name="tableHeader"
                size="mini"
                @selection-change="handleSelectionChange"
                row-key="tempId"
                ref="tb"
                max-height="380px"
            >
                <el-table-column type="selection" width="55"></el-table-column>
                <el-table-column type="index" width="55" label="序号"></el-table-column>

                <el-table-column label="字段">
                    <template slot-scope="scope">
                        <el-select
                            v-model="scope.row.sortFields"
                            placeholder="请选择字段"
                            size="mini"
                            filterable
                            clearable
                        >
                            <el-option
                                v-for="(value, key, index) in sortFieldsObj"
                                :key="index"
                                :label="value"
                                :value="key"
                                :disabled="chooseFields.includes(key)"
                            >
                            </el-option>
                        </el-select>
                    </template>
                </el-table-column>
                <el-table-column label="排序方式">
                    <template slot-scope="scope">
                        <el-select v-model="scope.row.sortMethod" placeholder="请选择字段" size="mini" filterable>
                            <el-option label="升序" value="asc"> </el-option>
                            <el-option label="降序" value="desc"> </el-option>
                        </el-select>
                    </template>
                </el-table-column>
            </el-table>
            <div slot="footer" class="dialog-footer">
                <el-button @click="cancelMethod" size="mini" w-65>取 消</el-button>
                <el-button type="primary" @click="confirmMethod" size="mini" w-65>确 定</el-button>
            </div>
        </el-dialog>
    </div>
</template>
<script>
    import { getSorts } from '@/utils/basic'
    export default {
        name: 'cunstomSort',
        props: {
            sortVisible: {
                type: Boolean,
                default: false,
            },
            flag: {
                type: String,
                default: '',
            },
            tableHeaderList: {
                type: Array,
                default: () => [],
            },
        },
        data() {
            return {
                sortFieldsObj: {},
                chooseRowIds: '',
                idx: 1,
                sortData: [],
                chooseFields: [],
            }
        },
        watch: {
            sortVisible(bool) {
                // console.log('cunstomSort-sortVisible: ', bool)
                if (bool) {
                    let _sortObj = {}
                    this.tableHeaderList.forEach((item) => {
                        _sortObj[item.prop] = item.label
                    })
                    getSorts(this.flag, _sortObj).then((res) => {
                        this.sortFieldsObj = res
                    })
                    if (!this.sortData.length) {
                        this.idx = 1
                        this.sortData.push({ sortFields: '', sortMethod: 'asc', tempId: this.idx })
                    } else {
                        this.sortData.forEach((item) => (item.tempId = ++this.idx))
                    }
                }
            },
            sortData: {
                handler(sortFields) {
                    this.chooseFields = []
                    sortFields.forEach((item) => {
                        if (item.sortFields && !this.chooseFields.includes(item.sortFields))
                            this.chooseFields.push(item.sortFields)
                    })
                },
                deep: true,
            },
        },
        methods: {
            // 单选框选中数据
            handleSelectionChange(selection) {
                this.chooseRowIds = selection.length ? selection.map((item) => item.tempId) : []
            },
            insertRow(type) {
                if (this.chooseRowIds.length > 1) {
                    this.$message.error('请选择一条数据进行插入')
                    return
                }
                // type === 'head' 插入行 type === 'behind' 增加行
                for (let index = 0; index < this.sortData.length; index++) {
                    const element = this.sortData[index]
                    if (this.chooseRowIds.includes(element.tempId)) {
                        this.sortData.splice(type === 'head' ? index : index + 1, 0, {
                            sortFields: '',
                            sortMethod: 'asc',
                            tempId: ++this.idx,
                        })
                        return
                    }
                }
            },
            deleteRow() {
                this.chooseRowIds.forEach((item) => {
                    for (let index = this.sortData.length - 1; index >= 0; index--) {
                        let element = this.sortData[index]
                        if (item === element.tempId) {
                            this.sortData.splice(index, 1)
                            return
                        }
                    }
                })
                if (!this.sortData.length) {
                    this.idx = 1
                    this.sortData.push({ sortFields: '', sortMethod: 'asc', tempId: this.idx })
                }
            },
            cancelMethod() {
                this.chooseRowIds = []
                this.$emit('close-customSort')
            },
            confirmMethod() {
                for (let index = 0; index < this.sortData.length; index++) {
                    let element = this.sortData[index]
                    if (!element.sortFields) {
                        this.$message.error(`请选择第${index + 1}行的字段`)
                        return
                    }
                }
                let arr = []
                this.sortData.forEach((item) => arr.push(item.sortFields + ' ' + item.sortMethod))
                this.$refs.tb.clearSelection()
                this.$emit('submit-customSort', arr.join(','))
            },
        },
    }
</script>
<style lang="less" scoped>
    @import url('../../pages/fims/common.less');
    .cunstom-sort {
        /deep/.el-dialog .el-dialog__body {
            padding: 10px 25px 0px 25px;
            .el-table {
                margin-top: 10px;
            }

            .el-button {
                margin-left: 0 !important;
            }
        }
    }
</style>
