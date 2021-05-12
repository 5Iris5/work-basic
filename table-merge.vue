<template>
    <div class="content-box">
        <div class="table-box" ref="tableBox">
            <div class="table-btn">
                <el-button @click="deleteOne('some')" :disabled="selList.length === 0" size="mini" w-90
                    >批量选择</el-button
                >
            </div>
            <div class="table-content">
                <el-table
                    stripe
                    :data="tableDataList"
                    :span-method="SpanMethod"
                    @selection-change="selectChange"
                    border
                    highlight-current-row
                    header-cell-class-name="tableHeader"
                    size="mini"
                    height="300px"
                    max-height="500px"
                    :cell-style="changeCellStyle"
                    @select="handleSelect"
                >
                    <el-table-column type="selection"></el-table-column>
                    <el-table-column prop="voucherNumber" label="字号" width="90">
                        <template slot-scope="scope">
                            <el-link type="primary" :underline="false">{{ scope.row.voucherNumber }}</el-link>
                        </template>
                    </el-table-column>
                    <el-table-column prop="businessDate" label="日期" width="90"> </el-table-column>
                    <el-table-column prop="brief" label="摘要" width="210"> </el-table-column>
                    <el-table-column prop="accountSubject" label="科目" width="180"> </el-table-column>
                    <el-table-column prop="trsamtd" label="借方金额" width="150" :formatter="formatterAmount">
                    </el-table-column>
                    <el-table-column prop="trsamtc" label="贷方金额" width="150" :formatter="formatterAmount">
                    </el-table-column>
                </el-table>
            </div>
        </div>
    </div>
</template>

<script>
    import { parseFormatNum } from '@/utils/index'
    export default {
        data() {
            return {
                ckedIds: [],
                tableData: [
                    {
                        voucherId: 100,
                        businessDate: '2020-11-16',
                        voucherNumber: 1,
                        entryInfo: [
                            {
                                brief: '摘要一',
                                urrId: 2059,
                                accountSubject: '其他货币资金',
                                trsamtd: 200,
                                trsamtc: 0,
                            },
                            {
                                brief: '摘要二',
                                urrId: 2060,
                                accountSubject: '应收股利',
                                trsamtd: 0,
                                trsamtc: 120,
                            },
                            {
                                brief: '摘要三',
                                urrId: 2061,
                                accountSubject: '未实现融资收益',
                                trsamtd: 0,
                                trsamtc: 80,
                            },
                        ],
                        createZh: 'Iris',
                        checkZh: 'Iris',
                        status: 2,
                    },
                    {
                        voucherId: 101,
                        businessDate: '2020-11-17',
                        voucherNumber: 2,
                        entryInfo: [
                            {
                                brief: '摘要一',
                                urrId: 2070,
                                accountSubject: '其他货币资金',
                                trsamtd: 100,
                                trsamtc: 0,
                            },
                        ],
                        createZh: 'Iris',
                        checkZh: 'Iris',
                        status: 1,
                    },
                    {
                        voucherId: 102,
                        businessDate: '2020-11-18',
                        voucherNumber: 3,
                        entryInfo: [
                            {
                                brief: '摘要一',
                                urrId: 2059,
                                accountSubject: '其他货币资金',
                                trsamtd: 200,
                                trsamtc: 0,
                            },
                            {
                                brief: '摘要二',
                                urrId: 2060,
                                accountSubject: '应收股利',
                                trsamtd: 0,
                                trsamtc: 200,
                            },
                        ],
                        createZh: 'Iris',
                        checkZh: 'Iris',
                        status: 2,
                    },
                ],
                tableDataList: [], // 重构后的数据结构
                spanArray: [],
                selList: [],
            }
        },
        mounted() {
            this.getIndex(this.tableData)
        },
        methods: {
            getIndex(tableDataList) {
                let arr = [] // 保存表格合并需要的数据
                let s = 0 // 合并初始位置
                let getTable = [] // 把原始数据中的子数据提取出来，生成新的可合并数据
                tableDataList.forEach((item, i, data) => {
                    // console.log(item)
                    // 第二次合并开始位置
                    if (arr.length) {
                        s = arr[arr.length - 1].row + data[i - 1].entryInfo.length
                    }
                    arr.push({
                        row: s, // 保存表格合并的起始行
                        index: item.entryInfo.length, // 每次需要合并几行
                    })
                    if (item.entryInfo && item.entryInfo.length)
                        item.entryInfo.forEach((subItem) => {
                            getTable.push({
                                voucherId: item.voucherId,
                                voucherNumber: item.voucherNumber,
                                businessDate: item.businessDate,
                                brief: subItem.brief,
                                accountSubject: subItem.accountSubject,
                                urrId: subItem.urrId,
                                trsamtd: subItem.trsamtd,
                                trsamtc: subItem.trsamtc,
                            })
                        })
                })
                this.spanArray = arr
                this.tableDataList = getTable
            },
            // 合并表格行
            SpanMethod(data) {
                let { rowIndex, column } = data
                let _props = ['businessDate', 'voucherNumber']
                if (!column.property || _props.includes(column.property)) {
                    let obj = [0, 0]
                    this.spanArray.some((v) => {
                        if (rowIndex === v.row) {
                            obj = [v.index, 1]
                        }
                    })
                    return obj
                }
            },

            deleteOne() {
                this.$confirm(`当前选中 ${this.selList.length} 条数据`, '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning',
                })
                    .then(() => {
                        this.$message.success('操作成功')
                    })
                    .catch(() => {
                        this.$message.info('已取消操作')
                    })
            },

            // 当勾选项发生变化时
            selectChange(rows) {
                let selList = rows.map((item) => item.voucherId)
                this.selList = selList
                    .filter((item, idx) => selList.indexOf(item, 0) === idx)
                    .filter((v) => this.ckedIds.indexOf(v) === -1)
            },
            // 获取取消勾选的数据
            handleSelect(selection, current) {
                let selected = selection.length && selection.indexOf(current) !== -1
                !selected
                    ? this.ckedIds.push(current.voucherId)
                    : this.ckedIds.splice(this.ckedIds.indexOf(current.voucherId), 1)
            },

            formatterAmount(row, column) {
                let _amount = row[column.property]
                return _amount ? parseFormatNum(Number(_amount), 2) : ''
            },
            changeCellStyle(row) {
                let _txt = ['借方金额', '贷方金额']
                if (_txt.includes(row.column.label)) {
                    return 'textAlign: right'
                }
            },
        },
    }
</script>

<style lang="less" scoped>
    @import url('../common.less');
    .content-box {
        .content {
            /deep/ .el-form {
                .el-row {
                    .sub-width {
                        .el-input {
                            width: 260px;
                        }
                    }
                }
            }
            .vouDur {
                display: inline-block;
                span {
                    display: inline-block;
                    line-height: 42px;
                }
            }
        }
    }
</style>
