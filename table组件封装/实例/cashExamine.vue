<template>
    <div class="content-box" ref="contentBox">
        <div :class="showSearch ? 'content wrap' : 'content'" ref="searchBox">
            <el-form :model="searchForm" inline ref="searchForm">
                <el-row>
                    <el-form-item v-show="searches.includes('accountBooksId')">
                        <el-select
                            v-model.number="searchForm.accountBooksId"
                            placeholder="帐套"
                            class="filter-item"
                            size="mini"
                            filterable
                        >
                            <el-option
                                v-for="item in accountBookList"
                                :key="item.accountBooksId"
                                :label="item.compayShortName"
                                :value="item.accountBooksId"
                            ></el-option>
                        </el-select>
                    </el-form-item>
                    <div class="vouDur" v-show="searches.includes('periodIds')">
                        <el-form-item>
                            <el-select
                                v-model="startPeriodId"
                                placeholder="会计期间"
                                class="filter-item"
                                size="mini"
                                clearable
                            >
                                <el-option
                                    v-for="item in vouDurList"
                                    :key="item.periodId"
                                    :label="item.fullValue"
                                    :value="item.periodId"
                                ></el-option>
                            </el-select>
                        </el-form-item>
                        <span>至</span>
                        <el-form-item>
                            <el-select
                                v-model="endPeriodId"
                                placeholder="会计期间"
                                class="filter-item"
                                size="mini"
                                clearable
                            >
                                <el-option
                                    v-for="item in vouDurList"
                                    :key="item.periodId"
                                    :label="item.fullValue"
                                    :value="item.periodId"
                                ></el-option>
                            </el-select>
                        </el-form-item>
                    </div>
                    <el-form-item v-show="searches.includes('cashStatus')">
                        <el-select
                            v-model.number="searchForm.cashStatus"
                            placeholder="是否指定现金流"
                            class="filter-item"
                            size="mini"
                            clearable
                        >
                            <el-option
                                v-for="item in chooseCashList"
                                :key="item.value"
                                :label="item.label"
                                :value="item.value"
                            ></el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item class="vouDur" v-show="searches.includes('currencyId')">
                        <el-select v-model.number="searchForm.currencyId" placeholder="币别" size="mini" clearable>
                            <el-option
                                v-for="item in currencyList"
                                :key="item.id"
                                :label="item.nameZh"
                                :value="item.id"
                            ></el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item v-show="searches.includes('itemId')">
                        <el-select
                            v-model.number="searchForm.itemId"
                            placeholder="项目查询"
                            class="filter-item"
                            size="mini"
                            clearable
                            filterable
                        >
                            <el-option
                                v-for="(item, index) in subCashList"
                                :key="index"
                                :label="`${item.cashCode} ${item.assisTypeName}`"
                                :value="item.id"
                            ></el-option>
                        </el-select>
                    </el-form-item>
                </el-row>
                <div class="btn-list">
                    <el-button @click="getList('isSearch')" type="primary" size="mini" w-90>查询</el-button>
                    <el-button @click="resetMethod" size="mini" w-90>重置</el-button>
                    <i
                        :class="['iconfont icon-pulldown', showSearch ? 'pullup' : '']"
                        @click="changeHeight"
                        v-if="showClose"
                    ></i>
                </div>
            </el-form>
        </div>
        <div class="table-box" ref="tableBox">
            <div class="table-btn">
                <el-button
                    v-show="buttons.includes('现金流')"
                    :disabled="selList.length !== 1"
                    @click="editCash('one')"
                    type="primary"
                    size="mini"
                    >现金流</el-button
                >
                <span
                    v-show="Boolean(columnList.length)"
                    v-for="item in cashAmountList"
                    :key="item.currencyId"
                    style="margin-right: 10px"
                    >{{ item.currencyZh }}: {{ formatterAmount(item.totalAmount) }}</span
                >
                <el-button
                    v-show="buttons.includes('自定义列')"
                    size="mini"
                    icon="el-icon-s-operation"
                    class="fr"
                    @click="openCustomColumn"
                    >自定义列</el-button
                >
                <el-button
                    v-show="buttons.includes('排序')"
                    size="mini"
                    icon="el-icon-d-caret"
                    class="fr"
                    style="margin-right: 5px"
                    :disabled="tableData.list.length === 0"
                    @click="openSort"
                    >排序</el-button
                >
                <el-button
                    v-show="buttons.includes('导出')"
                    size="mini"
                    icon="el-icon-download"
                    class="fr"
                    style="margin-right: 5px"
                    :disabled="tableData.list.length === 0"
                    @click="exportFile"
                    >导出</el-button
                >
            </div>
            <div class="table-content">
                <uTable
                    v-if="Boolean(columnList.length)"
                    ref="umyTable"
                    :rowkey="'urrId'"
                    :height="height"
                    :tableData="tableData"
                    :columnList="columnList"
                    :textAlignRightList="textAlignRightList"
                    :pageNumber="pageNumber"
                    :pageSize="pageSize"
                    :showCustomColumn="showCustomColumn"
                    :viewId="viewId"
                    :tableId="tableId"
                    :showCustomSort="showCustomSort"
                    :flag="'StatVoucherinfoEntity'"
                    @go-page="goPage"
                    @change-size="changeSize"
                    @selection-change="selectChange"
                    @get-tableProps-by-customColumn="getTableProps"
                    @submit-sort="submitCustomSort"
                    @reset-tableProps-sort="resetPropSort"
                >
                    <template slot="normal" slot-scope="scope">
                        <template v-if="scope.row.prop === 'voucherNumber'">
                            <span class="specialcolor" @click="editOne('cashexamine', scope.row.data)">{{
                                scope.row.data.voucherNumber
                            }}</span>
                        </template>
                        <template v-else-if="textAlignRightList.includes(scope.row.prop)">
                            <span v-if="scope.row.prop === 'trsamtd'">{{
                                scope.row.data[scope.row.prop]
                                    ? formatterAmount(scope.row.data.trsamtd)
                                    : formatterAmount(scope.row.data.trsamtc)
                            }}</span>
                            <span v-else-if="scope.row.prop === 'trsamtc'">{{
                                scope.row.data.conTrsamtc
                                    ? formatterAmount(scope.row.data.conTrsamtc)
                                    : formatterAmount(scope.row.data.conTrsamtd)
                            }}</span>
                            <span v-else>{{ formatterAmount(scope.row.data[scope.row.prop]) }}</span>
                        </template>
                        <template v-else-if="scope.row.prop === 'currencyId'">
                            <span>{{ formatterCurrency(scope.row.data[scope.row.prop]) }}</span>
                        </template>
                        <template v-else-if="scope.row.prop === 'itemName'">
                            <span class="specialcolor" @click="editCash('detail', scope.row.data)">
                                {{ scope.row.data.itemName }}</span
                            >
                        </template>
                        <template v-else
                            ><span>{{ scope.row.data[scope.row.prop] }}</span></template
                        >
                    </template>
                </uTable>
                <div v-else class="no-roles">
                    <img src="../../../assets/images/role.png" alt="暂无此页面权限" />
                    <p>暂无此页面权限!</p>
                </div>
            </div>
        </div>
        <exitCash
            :showExitCash="showExitCash"
            :exitCashList="exitCashList"
            :nonExitCashList="nonExitCashList"
            :subCashList="subCashList"
            @cancelExitCashDialog="cancelExitCashDialog"
            @cofirmExitCashDialog="cofirmExitCashDialog"
            :isNormalShow="isNormalShow"
        ></exitCash>
    </div>
</template>

<script>
    import uTable from '@/components/view/umyTable'
    import { findView } from '@/api/system/menu'
    import exitCash from '@/components/voucher/exitCashSelect'
    import { findAllAccountBook } from '@/api/fims/setSubject'
    import { findAllCurrency } from '@/api/fims/setPeriod'
    import { findVouDur, findAllVou, cashFlowDesignated, prestoreCashFlow, dropDownList } from '@/api/fims/voucherSelect'
    import { unique, parseFormatNum, deepCopy, resetColProps } from '@/utils/index'
    import { exportExcels } from '@/api/fims/common'

    export default {
        name: 'cashExamine',
        components: { exitCash, uTable },
        computed: {
            periodIds() {
                return [this.startPeriodId, this.endPeriodId]
            },
        },
        data() {
            return {
                height: 0,
                searches: [],
                buttons: [],
                tableOperate: [],
                columnList: [],
                showCustomColumn: false,
                viewId: null,
                tableId: null,
                showCustomSort: false,
                sortParams: '',
                textAlignRightList: ['trsamtd', 'conTrsamtd', 'conTrsamtc', 'cashAmount'],
                spanFiled: ['businessDate', 'voucherNumber'],

                searchForm: {},
                accountBookList: [],
                vouDurList: [],
                currencyList: [],
                tableData: {
                    list: [],
                },
                tableDataList: [], // 重构后的数据结构
                selList: [],
                pageSize: 100,
                pageNumber: 1,
                initEnter: false,
                showSearch: false,
                showClose: false,
                contentDom: '',
                isNormalShow: true,
                showExitCash: false,
                exitCashList: [],
                nonExitCashList: [],
                subCashList: [],
                startPeriodId: '',
                endPeriodId: '',
                chooseCashList: [
                    {
                        label: '全部',
                        value: 1,
                    },
                    {
                        label: '已指定',
                        value: 2,
                    },
                    {
                        label: '未指定',
                        value: 3,
                    },
                ],
                cashItem: [],
                cashAmountList: [],
            }
        },
        created() {
            let { name } = this.$route
            this.getView(name)
        },
        mounted() {
            this.initEnter = true
            this.getInit()
            this.getInitHeight()
        },
        activated() {
            this.initEnter ? (this.initEnter = false) : this.getList()
        },
        methods: {
            getInit() {
                this.getAllCurrency()
                this.getSubCashList()
                this.getAllAccountBook()
            },
            async getAllCurrency() {
                let { data } = await findAllCurrency({
                    pageNo: 1,
                    pageSize: 100,
                })
                data.ret === 1 ? (this.currencyList = data.data.list) : this.$message.error(data.msg)
            },
            changeAccountBook(val) {
                this.getVouDur(val)
            },
            async getAllAccountBook() {
                let { data } = await findAllAccountBook({
                    pageNo: 1,
                    pageSize: 200,
                })
                if (data.ret === 1) {
                    this.accountBookList = data.data.list
                    this.$set(this.searchForm, 'accountBooksId', this.accountBookList[0].accountBooksId)
                    this.getVouDur(this.searchForm.accountBooksId)
                } else {
                    this.$message.error(data.msg)
                }
            },
            async getVouDur(accountBooksId) {
                let { data } = await findVouDur({
                    pageNo: 1,
                    pageSize: 100,
                    accountBooksId,
                })
                if (data.ret === 1) {
                    this.vouDurList = data.data.list
                    this.vouDurList.forEach((item) => {
                        if (item.isCurrentPeriod === 1) {
                            this.startPeriodId = this.endPeriodId = item.periodId
                        }
                    })
                } else {
                    this.$message.error(data.msg)
                }
                this.getList()
            },
            // 辅助核算现金流
            async getSubCashList() {
                let { data } = await dropDownList({
                    assisTypeId: 6,
                })
                data.ret === 1 ? (this.subCashList = data.data) : this.$message.error(data.msg)
            },

            //现金流
            async editCash(type, row) {
                // console.log(type, row)
                let _cashkey = type === 'one' ? `${this.cashItem[0].voucherKey}` : `${row.voucherKey}`
                let { data } =
                    type === 'one'
                        ? await cashFlowDesignated({
                              voucherId: this.selList[0],
                          })
                        : await cashFlowDesignated({
                              voucherId: row.voucherId,
                          })
                if (data.ret === 1) {
                    let _data = data.data.data
                    this.key = `${_cashkey}_cash`
                    this.exitCashList = _data.cashList.map((item) => {
                        let key = 'only'
                        let value = `${item.cashAccountId}_${item.currencyId}`
                        item[key] = value
                        if (item.nonCashflowList.length) {
                            let nonCashflowList = item.nonCashflowList
                            nonCashflowList.map((nTtem) => {
                                nTtem.cashAccountId = `${nTtem.cashAccountId}-${nTtem.vouEntryKey}`
                                return nTtem
                            })
                        }
                        return item
                    })
                    this.nonExitCashList = _data.nonCashList
                    // console.log('对方总科目分录: ', this.nonExitCashList)
                    this.$nextTick(() => {
                        this.showExitCash = true
                        this.getSubCashList()
                    })
                } else {
                    this.$message.error(data.msg)
                }
            },
            cancelExitCashDialog() {
                this.showExitCash = false
            },
            cofirmExitCashDialog() {
                let exitCashList = deepCopy(this.exitCashList)
                exitCashList.map((item) => {
                    if (item.nonCashflowList.length) {
                        let nonCashflowList = item.nonCashflowList
                        nonCashflowList.map((nTtem) => {
                            nTtem.cashAccountId = nTtem.cashAccountId.substring(0, nTtem.cashAccountId.indexOf('-'))
                            nTtem.cashAccountId = Number(nTtem.cashAccountId)
                            return nTtem
                        })
                        return item
                    }
                })
                let saveaddCashData = {
                    cashflowinfoList: exitCashList,
                    key: this.key,
                    isSave: true,
                    isAdjust: true,
                }
                this.$nextTick(() => {
                    this.prestoreCashFlow(saveaddCashData)
                })
                // console.log(saveaddCashData)
            },
            async prestoreCashFlow(saveaddCashData) {
                let { data } = await prestoreCashFlow({
                    ...saveaddCashData,
                })
                if (data.ret === 1) {
                    this.$message.success(data.msg)
                    this.showExitCash = false
                    this.getList()
                } else {
                    this.$message.error(data.msg)
                }
            },
            editOne(type, row) {
                sessionStorage.setItem('setVoucherTempData', JSON.stringify({}))
                sessionStorage.setItem('changeIsInitEnter', false)
                this.$nextTick(() => {
                    let parmas = {
                        type: type,
                        voucherId: row.voucherId,
                    }
                    sessionStorage.setItem('setVoucherTempData', JSON.stringify(parmas))
                    this.$router.push('/fims/voucher/newly')
                })
            },

            async getList(search) {
                if (search && search !== 'GoPage') this.pageNumber = 1
                let _params = {
                    pageNo: this.pageNumber,
                    pageSize: this.pageSize,
                    isCashFlow: 1,
                    ...this.searchForm,
                    orderParams: this.sortParams,
                }
                let { data } =
                    this.startPeriodId && this.endPeriodId
                        ? await findAllVou({ periodIds: this.periodIds, ..._params })
                        : await findAllVou({ ..._params })
                if (data.ret === 1) {
                    this.tableData = data.data
                    this.cashAmountList = data.data.obj || []
                    this.tableDataList =
                        data.data && data.data.list.length ? this.flatTableList(data.data.list, this.spanFiled) : []
                    this.$nextTick(() => {
                        if (this.columnList.length) {
                            this.$refs.umyTable.$refs.uTable.reloadData(this.tableDataList)
                            this.$refs.umyTable.$refs.uTable.doLayout()
                        }
                    })
                } else {
                    this.$message.error(data.msg)
                }
            },
            flatTableList(tableDataList, spanFiled) {
                let getTable = []
                tableDataList.forEach((item) => {
                    if (item.entryInfo && item.entryInfo.length)
                        item.entryInfo.forEach((subItem, subIdx) => {
                            let _info = {
                                accountBooksId: item.accountBooksId,
                                voucherId: item.voucherId,
                                businessDate: item.businessDate,
                                voucherNumber: item.voucherNumber,
                                brief: subItem.brief,
                                urrId: subItem.urrId,
                                urrId2: subItem.urrId2,
                                accountSubject: subItem.accountSubject,
                                currencyId: subItem.currencyId,
                                trsamtd: subItem.trsamtd,
                                trsamtc: subItem.trsamtc,
                                conTrsamtd: subItem.conTrsamtd,
                                conTrsamtc: subItem.conTrsamtc,
                                voucherKey: item.voucherKey,
                            }
                            if (subItem.cashVoucherList && subItem.cashVoucherList.length) {
                                subItem.cashVoucherList.forEach((cashItem, cashIdx) => {
                                    if (cashIdx === 0) {
                                        _info = this.setSpanstyle(subIdx, { ..._info, ...cashItem }, spanFiled)
                                        getTable.push(_info)
                                    } else {
                                        cashItem.hideSelection = true
                                        getTable.push({
                                            ...cashItem,
                                            voucherKey: item.voucherKey,
                                            voucherId: item.voucherId,
                                        })
                                    }
                                })
                            } else {
                                _info = this.setSpanstyle(subIdx, _info, spanFiled)
                                getTable.push(_info)
                            }
                        })
                })
                return getTable
            },
            setSpanstyle(subIdx, _info, spanFiled) {
                if (subIdx) {
                    for (const key in _info) {
                        if (spanFiled.includes(key)) _info[key] = ''
                    }
                }
                _info.hideSelection = subIdx ? true : false
                return _info
            },

            formatterAmount(value) {
                return value ? parseFormatNum(Number(value), 2) : ''
            },
            formatterCurrency(val) {
                let currency = this.currencyList.filter((item) => item.id === val)
                if (currency.length !== 0) return currency[0].nameZh
            },
            selectChange(rows) {
                let selList = rows.map((item) => item.voucherId)
                this.selList = unique(selList)
                this.cashItem = rows.map((item) => item)
                // console.log('凭证voucherId： ', this.selList, rows)
            },

            changeSize(pageSize) {
                this.pageSize = pageSize
                this.getList('search')
            },
            goPage(pageNo) {
                this.pageNumber = pageNo
                this.getList('GoPage')
            },
            resetMethod() {
                this.searchForm = {}
                this.startPeriodId = this.endPeriodId = ''
                this.getList('search')
            },

            async getView(perms) {
                let { data } = await findView({ perms })
                if (data.ret === 1) {
                    let _data = data.data[0] || [],
                        _searches = _data.searches || [],
                        _buttons = _data.buttons || [],
                        _tables = _data.tables[0] || [],
                        _tableOperate = _tables.buttons || []
                    this.searches = _searches.map((v) => v.value)
                    this.buttons = _buttons.map((v) => v.name)
                    this.tableOperate = _tableOperate.map((v) => v.name)
                    this.columnList = _tables.fields.sort((a, b) => a.sort - b.sort)
                    this.viewId = _tables.viewId
                    this.tableId = _tables.id
                    this.showDetailBtnFun()
                    this.$calUmyTableMethods(this.calcMethods)
                } else {
                    this.$message.error(data.msg)
                }
            },
            openCustomColumn() {
                this.showCustomColumn = true
            },
            getTableProps(tableProps) {
                this.columnList = tableProps
            },
            openSort() {
                this.showCustomSort = true
            },
            submitCustomSort(data) {
                this.sortParams = data
                this.showCustomSort = false
                this.getList()
            },
            resetPropSort(tableProps, oldIndex, newIndex) {
                this.columnList = resetColProps(tableProps, oldIndex, newIndex)
            },
            exportFile() {
                let data = {
                    searchData: {
                        ...this.searchForm,
                        periodIds: this.periodIds[0] ? this.periodIds : null,
                        isCashFlow: 1,
                    },
                    excleName: this.$route.meta.title,
                    url: '/api/1/fmis/statVoucherinfo/find-all',
                    exportExcelsMethods: exportExcels,
                    dataNumOrTitle: this.$refs.umyTable.getCustomExcels(),
                }
                this.$exportTableData(data)
            },

            changeHeight() {
                this.showSearch = !this.showSearch
                this.$nextTick(() => this.calcMethods())
            },
            calcMethods() {
                let allH = this.$refs.contentBox.clientHeight
                let searchH = this.$refs.searchBox.clientHeight
                this.height = allH - searchH - 100
            },
            getInitHeight() {
                window.addEventListener('resize', this.$calcTableHeight())
                this.contentDom = this.$refs.searchBox
                window.addEventListener('resize', this.showDetailBtnFun)
            },
            showDetailBtnFun() {
                this.$nextTick(() => {
                    this.showSearch = false
                    this.showClose = 50 < this.contentDom.scrollHeight ? true : false
                })
            },
        },
        beforeDestroy() {
            window.removeEventListener('resize', this.$calcTableHeight())
            window.removeEventListener('resize', this.showDetailBtnFun)
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
