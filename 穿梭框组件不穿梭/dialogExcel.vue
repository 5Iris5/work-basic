<template>
    <div class="dialog-excel">
        <el-dialog :title="dialogTitle" width="60%" :visible.sync="excelVisible" :before-close="cancelMethod">
            <div class="table-content">
                <div class="init-table">
                    <el-table
                        ref="initTable"
                        :data="tableData"
                        size="mini"
                        border
                        height="160px"
                        max-height="160px"
                        @current-change="(e) => handleChange(e, 'init')"
                        highlight-current-row
                    >
                        <el-table-column
                            v-for="item in tableTit"
                            :key="item.prop"
                            :prop="item.prop"
                            :label="item.label"
                        >
                        </el-table-column>
                    </el-table>
                </div>
                <div class="table-btn">
                    <el-button size="mini" @click="(e) => handleCurrentRow(e, 'add')" :disabled="addDisabled"
                        >增加</el-button
                    >
                    <el-button size="mini" @click="(e) => handleCurrentRow(e, 'del')" :disabled="delDisabled"
                        >删除</el-button
                    >
                </div>
                <div class="excel-table">
                    <el-table
                        ref="excelTable"
                        :data="excelData"
                        size="mini"
                        border
                        height="160px"
                        max-height="160px"
                        @current-change="(e) => handleChange(e, 'create')"
                        highlight-current-row
                    >
                        <el-table-column
                            v-for="item in tableTit"
                            :key="item.prop"
                            :prop="item.prop"
                            :label="item.label"
                        >
                        </el-table-column>
                    </el-table>
                </div>
            </div>
            <div slot="footer" class="dialog-footer">
                <el-button @click="cancelMethod" size="mini" w-65>取 消</el-button>
                <el-button type="primary" @click="confirmMethod" size="mini" w-65>确 定</el-button>
            </div>
        </el-dialog>
    </div>
</template>

<script>
    export default {
        name: 'dialogExcel',
        props: {
            excelVisible: {
                type: Boolean,
                default: false,
            },
            dialogTitle: {
                type: String,
                default: '',
            },
        },
        data() {
            return {
                addDisabled: true,
                delDisabled: true,
                tableTit: [
                    {
                        prop: 'code',
                        label: '科目代码',
                    },
                    {
                        prop: 'name',
                        label: '科目名称',
                    },
                ],
                tableData: [
                    {
                        id: 1,
                        code: '2016-05-02',
                        name: '王小虎',
                    },
                    {
                        id: 2,
                        code: '2016-05-04',
                        name: '王小虎',
                    },
                    {
                        id: 3,
                        code: '2016-05-01',
                        name: '王小虎',
                    },
                    {
                        id: 4,
                        code: '2016-05-03',
                        name: '王小虎',
                    },
                    {
                        id: 5,
                        code: '2016-05-04',
                        name: '王小虎',
                    },
                    {
                        id: 6,
                        code: '2016-05-01',
                        name: '王小虎',
                    },
                    {
                        id: 7,
                        code: '2016-05-03',
                        name: '王小虎',
                    },
                ],
                excelTit: [],
                excelData: [],
                curRow: null,
            }
        },
        methods: {
            cancelMethod() {
                this.$emit('cancel-excel-method')
            },
            confirmMethod() {
                let data = '从子级传递给父级的数据...'
                this.$emit('confirm-excel-method', data)
            },

            // 左边原始table -> 取消目前高亮行的选中状态
            cancelInitRow() {
                this.$refs.initTable.setCurrentRow()
                this.addDisabled = true
            },
            // 右边生成table -> 取消目前高亮行的选中状态
            cancelCreateRow() {
                this.$refs.excelTable.setCurrentRow()
                this.delDisabled = true
            },

            // 选中行
            handleChange(curRow, type) {
                if (!curRow) return
                this.curRow = curRow
                if (type === 'init') {
                    this.addDisabled = false
                    this.cancelCreateRow()
                } else {
                    this.delDisabled = false
                    this.cancelInitRow()
                }
            },
            // 操作选中行
            handleCurrentRow(e, type) {
                if (type === 'add') {
                    this.excelData.push(this.curRow)
                    this.cancelInitRow()
                } else {
                    this.delDisabled = true
                    let idx = this.excelData.map((v) => v.id)
                    this.excelData.splice(idx.indexOf(this.curRow.id), 1)
                }
            },
        },
    }
</script>

<style lang="less" scoped>
    .dialog-excel {
        /deep/.el-dialog .el-dialog__body {
            padding: 10px 25px 0px 25px;
        }
        .table-content {
            width: 100%;
            .init-table {
                float: left;
            }
            .excel-table {
                float: right;
            }
            .table-btn {
                float: left;
                width: 40%;
                height: 160px;
                margin-top: 10px;
                .el-button {
                    display: block;
                    margin: 0 auto;
                    &:nth-of-type(1) {
                        margin-top: 45px;
                        margin-bottom: 5px;
                    }
                }
            }
            .init-table,
            .excel-table {
                width: 30%;
                margin-top: 10px;
                .el-table {
                    width: 100%;
                }
            }
            &::after {
                content: '';
                display: block;
                clear: both;
            }
        }
    }
</style>


