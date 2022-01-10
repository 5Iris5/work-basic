<template>
    <div class="content-box" ref="contentBox">
        <div :class="['searchBox', showSearch ? 'content wrap' : 'content']" ref="searchBox">
            <el-form :model="searchForm" inline ref="searchForm">
                <el-row>
                    <el-form-item v-show="searches.includes('accountBooksId')">
                        <el-select
                            v-model.number="searchForm.accountBooksId"
                            placeholder="帐套"
                            class="filter-item"
                            size="mini"
                            filterable
                            @change="changeAccountBook"
                        >
                            <el-option
                                v-for="item in accountBookList"
                                :key="item.accountBooksId"
                                :label="item.compayShortName"
                                :value="item.accountBooksId"
                            ></el-option>
                        </el-select>
                    </el-form-item>
                    <div class="vouDur" v-show="searches.includes('startPeriodId') && searches.includes('endPeriodId')">
                        <el-form-item v-show="searches.includes('startPeriodId')">
                            <el-select
                                v-model="searchForm.startPeriodId"
                                placeholder="会计期间"
                                class="filter-item"
                                size="mini"
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
                        <el-form-item v-show="searches.includes('endPeriodId')">
                            <el-select
                                v-model="searchForm.endPeriodId"
                                placeholder="会计期间"
                                class="filter-item"
                                size="mini"
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
                    <div
                        class="vouDur"
                        v-show="searches.includes('startSubjectCode') && searches.includes('endSubjectCode')"
                    >
                        <el-form-item v-show="searches.includes('startSubjectCode')">
                            <el-select
                                v-model="searchForm.startSubjectCode"
                                placeholder="起始科目"
                                class="filter-item"
                                size="mini"
                                filterable
                                clearable
                            >
                                <el-option
                                    v-for="item in subjectInfoList"
                                    :key="item.subjectCode"
                                    :label="item.initLabel"
                                    :value="item.subjectCode"
                                ></el-option>
                            </el-select>
                        </el-form-item>
                        <span>至</span>
                        <el-form-item v-show="searches.includes('endSubjectCode')">
                            <el-select
                                v-model="searchForm.endSubjectCode"
                                placeholder="结束科目"
                                class="filter-item"
                                size="mini"
                                filterable
                                clearable
                            >
                                <el-option
                                    v-for="item in subjectInfoList"
                                    :key="item.subjectCode"
                                    :label="item.initLabel"
                                    :value="item.subjectCode"
                                ></el-option>
                            </el-select>
                        </el-form-item>
                    </div>
                    <el-form-item v-show="searches.includes('subjectLever')">
                        <el-input-number
                            placeholder="科目级别"
                            v-model="searchForm.subjectLever"
                            controls-position="right"
                            :min="1"
                            :max="4"
                            size="mini"
                        ></el-input-number>
                    </el-form-item>
                    <el-form-item v-show="searches.includes('currencyId')">
                        <el-select v-model.number="searchForm.currencyId" placeholder="币别" size="mini">
                            <el-option
                                v-for="item in currencyList"
                                :key="item.id"
                                :label="item.nameZh"
                                :value="item.id"
                            ></el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item class="multiple-select-l" v-show="searches.includes('queryTypes')">
                        <el-select
                            v-model="searchForm.queryTypes"
                            placeholder="其他查询"
                            class="filter-item"
                            size="mini"
                            multiple
                            collapse-tags
                            clearable
                        >
                            <el-option
                                v-for="item in subList"
                                :key="item.value"
                                :label="item.label"
                                :value="item.value"
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
                <!-- <el-button
                    v-show="buttons.includes('自定义列')"
                    size="mini"
                    icon="el-icon-s-operation"
                    class="fr"
                    @click="openCustomColumn"
                    >自定义列</el-button
                > -->
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
                    :isChangeUI="true"
                    :height="height"
                    :columnList="columnList"
                    :textAlignRightList="textAlignRightList"
                    :colFixedList="colFixedList"
                    :treeNodeList="treeNodeList"
                    :showSelection="true"
                    :showPagination="false"
                    :showCustomColumn="showCustomColumn"
                    :viewId="viewId"
                    :tableId="tableId"
                    @get-tableProps-by-customColumn="getTableProps"
                >
                    <template slot="normal" slot-scope="scope">
                        <template v-if="scope.row.prop === 'subjectName'">
                            <span class="specialcolor" @click="detailOne(scope.row.data)">{{
                                scope.row.data.subjectName
                            }}</span>
                        </template>
                        <template v-else-if="textAlignRightList.includes(scope.row.prop)">
                            <span>{{ formatterAmount(scope.row.data[scope.row.prop]) }}</span>
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
    </div>
</template>
<script>
    import uTable from '@/components/view/umyTable'
    import { findView } from '@/api/system/menu'
    import { findAllAccountBook } from '@/api/fims/setSubject'
    import { findAllCurrency } from '@/api/fims/setPeriod'
    import { findVouDur } from '@/api/fims/voucherSelect'
    import { findAllSubject } from '@/api/fims/receiptExpense'
    import { findAll } from '@/api/fims/subjectBalance'
    import { sum, parseFormatNum } from '@/utils/index'
    import { exportExcels } from '@/api/fims/common'

    export default {
        name: 'subjectBalance',
        components: { uTable },
        data() {
            return {
                height: 0,
                searches: [],
                buttons: [],
                tables: [],
                columnList: [],
                showCustomColumn: false,
                viewId: null,
                tableId: null,
                textAlignRightList: [
                    'debitPeriodBeginBalance',
                    'conDebitPeriodBeginBalance',
                    'creditPeriodBeginBalance',
                    'conCreditPeriodBeginBalance',
                    'debitCurrPeriodBalance',
                    'conDebitCurrPeriodBalance',
                    'creditCurrPeriodBalance',
                    'conCreditCurrPeriodBalance',
                    'yearDebitBalance',
                    'conYearDebitBalance',
                    'yearCreditBalance',
                    'conYearCreditBalance',
                    'debitPeriodEndBalance',
                    'conDebitPeriodEndBalance',
                    'creditPeriodEndBalance',
                    'conCreditPeriodEndBalance',
                ],
                colFixedList: ['subjectCode', 'subjectName'],
                treeNodeList: ['subjectCode'],

                treeLists: [],
                // 帐套
                searchForm: {
                    accountBooksId: '',
                    startPeriodId: '',
                    endPeriodId: '',
                    startSubjectCode: '',
                    endSubjectCode: '',
                    subjectLever: 4,
                    currencyId: '',
                    queryTypes: '',
                },
                subList: [
                    {
                        value: 7,
                        label: '显示辅助核算',
                    },
                    {
                        value: 6,
                        label: '包括余额为0的科目',
                    },
                    // {
                    //     value: 5,
                    //     label: '包括没有业务发生的科目',
                    // },
                    // {
                    //     value: 4,
                    //     label: '包括本期没有发生额的科目',
                    // },
                    // {
                    //     value: 3,
                    //     label: '包括本年没有发生额的科目',
                    // },
                    // {
                    //     value: 2,
                    //     label: '包括未过账的凭证',
                    // },
                ],
                accountBookList: [],
                tableData: {
                    list: [],
                },
                selList: [],
                vouDurList: [],
                subjectInfoList: [],
                currencyList: [],
                // 搜索栏
                showSearch: false,
                showClose: false,
                contentDom: '',
                initEnter: false,
                standardCurrency: '',
                sum: [],
            }
        },
        created() {
            let { name } = this.$route
            this.getView(name)
        },
        mounted() {
            this.initEnter = true
            this.getAllAccountBook()
            this.getInitHeight()
        },
        activated() {
            this.initEnter ? (this.initEnter = false) : this.getList()
        },
        methods: {
            async getAllAccountBook() {
                let { data } = await findAllAccountBook({
                    pageNo: 1,
                    pageSize: 200,
                })
                if (data.ret === 1) {
                    this.accountBookList = data.data.list
                    this.$set(this.searchForm, 'accountBooksId', this.accountBookList[0].accountBooksId)
                    this.standardCurrency = this.accountBookList[0].currencyId
                    this.getAllSubject(this.searchForm.accountBooksId)
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
                            this.searchForm.startPeriodId = this.searchForm.endPeriodId = item.periodId
                        }
                    })
                } else {
                    this.$message.error(data.msg)
                }
            },
            changeAccountBook(value) {
                this.$set(this.searchForm, 'startSubjectCode', '')
                this.$set(this.searchForm, 'endSubjectCode', '')
                this.standardCurrency = this.accountBookList.filter((item) => item.accountBooksId === value)[0].currencyId
                this.getAllSubject(value)
            },
            async getAllSubject(accountBooksId) {
                let { data } = await findAllSubject({
                    pageNo: 1,
                    pageSize: 1000,
                    accountBooksId,
                    queryType: 1,
                })
                if (data.ret === 1) {
                    this.subjectInfoList = data.data.list.map((v) => {
                        if (v) {
                            let key = 'initLabel'
                            let value = `${v.subjectCode} ${v.fullName}`
                            v[key] = value
                        }
                        return v
                    })
                } else {
                    this.$message.error(data.msg)
                }
                this.getAllCurrency()
            },
            async getAllCurrency() {
                let { data } = await findAllCurrency({
                    pageNo: 1,
                    pageSize: 100,
                })
                if (data.ret === 1) {
                    data.data.list.forEach((item) => {
                        item.nameZh =
                            item.id === this.standardCurrency && item.nameZh.indexOf('本位币') < 0
                                ? item.nameZh + '(本位币)'
                                : item.nameZh
                    })
                    this.currencyList = [
                        { id: 98, nameZh: '综合本位币' },
                        { id: 99, nameZh: '所有币别' },
                        ...data.data.list,
                    ]
                    this.$set(this.searchForm, 'currencyId', this.currencyList[0].id)
                    this.getList()
                } else {
                    this.$message.error(data.msg)
                }
            },

            detailOne(row) {
                // console.log(row)
                let { subjectCode, subjectName, isSubject, id, parentId } = row
                let onlyOne = isSubject === 1 ? id : parentId
                let {
                    accountBooksId,
                    startPeriodId,
                    endPeriodId,
                    currencyId,
                    queryTypes,
                    startSubjectCode,
                    endSubjectCode,
                    subjectLever,
                } = this.searchForm
                let initLabel = `${subjectCode} ${subjectName}`.replace(/(^\s*)|(\s*$)/g, '')
                this.$nextTick(() => {
                    let parmas = {
                        accountBooksId,
                        startPeriodId,
                        endPeriodId,
                        currencyId: row.currencyId ? row.currencyId : currencyId,
                        onlyOne,
                        isSubject,
                        initLabel,
                        isShowAccountInfo: queryTypes.includes(7) ? 1 : 0,
                        isZero: queryTypes.includes(6) ? 1 : 0,
                        // isIncurred: queryTypes.includes(4) ? 1 : 0,
                        isIncurred: 1,
                        startCode: startSubjectCode,
                        endCode: endSubjectCode,
                        level: subjectLever,
                    }
                    sessionStorage.setItem('setSubBalance', JSON.stringify(parmas))
                    this.$router.push('/fims/accountBook/subDetailAcc')
                })
            },
            getCurrencyUi() {
                /**
                 * 展示列数:
                 *  1. 所有币别 99 / 综合本位币 98 / 本位币 -> 2列
                 *  2. 其他币别 -> 4列
                 */
                let oneCurrencies = [99, 98, this.standardCurrency],
                    { currencyId } = this.searchForm,
                    _columnList

                if (oneCurrencies.includes(currencyId)) {
                    _columnList = this.tables.filter((v) => v.tableKey === 'standardCurrency')[0]
                } else {
                    _columnList = this.tables.filter((v) => v.tableKey === 'otherCurrency')[0]
                }
                this.columnList = _columnList.fields.sort((a, b) => a.sort - b.sort)
                // console.log('currencyId: ', currencyId, _columnList, this.columnList)
                this.viewId = _columnList.viewId
                this.tableId = _columnList.id
            },
            async getList(search) {
                if (search) this.pageNumber = 1
                let obj = this.deleteEmptyParams()
                let { data } = await findAll({
                    pageNo: 1,
                    pageSize: 500,
                    ...obj,
                })
                if (data.ret === 1) {
                    this.sum = []
                    this.getCurrencyUi()
                    this.tableData = data.data
                    let tableDataList = data.data.list || []
                    if (tableDataList.length) {
                        /**
                         * 重构树形数据结构:
                         * 所有币别 99
                         *  1. 折合本位币 取有con的数据
                         *  2. 非折合本位币 取无con的数据 -> 正常取值
                         * 非本位币的单币别 !99 && !98 && !this.standardCurrency
                         *  1. 返回两条数据，一条为原币，一条为折合本位币，将两条数据构建成一条数据
                         *      1）原币 取无con的数据
                         *      2）本位币 取有con的数据
                         */
                        let oneCurrencies = [99, 98, this.standardCurrency]
                        if (this.searchForm.currencyId === 99) {
                            this.treeLists = this.resetTree(tableDataList, 'allCurrency')
                        } else if (!oneCurrencies.includes(this.searchForm.currencyId)) {
                            this.treeLists = this.resetTree(tableDataList, 'oneCurrency')
                        } else {
                            this.treeLists = this.resetTree(tableDataList, false)
                        }
                        if (this.searchForm.currencyId !== 99) this.getSummaries(this.treeLists)
                    } else {
                        this.treeLists = []
                    }
                    // console.log('treeLists: ', this.treeLists)
                    this.$nextTick(() => {
                        if (this.columnList.length) {
                            this.$refs.umyTable.$refs.uTable.reloadData(this.treeLists)
                            this.$refs.umyTable.$refs.uTable.doLayout()
                        }
                    })
                } else {
                    this.$message.error(data.msg)
                }
            },
            resetTree(initTable, type) {
                let treeLists = [] // 把amountChild里面的数据提出来
                initTable.forEach((item) => {
                    if (item.amountChild && item.amountChild.length) {
                        if (type === 'oneCurrency') {
                            // 单币别->外币 两条数据重构为一条数据
                            let firstItem = item.amountChild.filter((item) => item.currencyZh !== '折合本位币')[0]
                            let secondItem = item.amountChild.filter((item) => item.currencyZh === '折合本位币')[0]
                            // console.log(type, item.children)
                            treeLists.push({
                                ...this.getCommons(item),
                                children: this.resetTree(item.children, type),
                                ...this.setMergeAmount(firstItem, secondItem),
                            })
                        } else {
                            item.amountChild.forEach((aItem, aIndex) => {
                                treeLists.push({
                                    ...this.getCommons(item),
                                    children:
                                        aIndex === item.amountChild.length - 1 ? this.resetTree(item.children, type) : [],
                                    currencyId: aItem.currencyId ? aItem.currencyId : item.currencyId,
                                    currencyZh: aItem.currencyZh,
                                    conCreditCurrPeriodBalance: aItem.conCreditCurrPeriodBalance,
                                    conCreditPeriodBeginBalance: aItem.conCreditPeriodBeginBalance,
                                    conCreditPeriodEndBalance: aItem.conCreditPeriodEndBalance,
                                    conCreditYearBeginBalance: aItem.conCreditYearBeginBalance,
                                    conDebitCurrPeriodBalance: aItem.conDebitCurrPeriodBalance,
                                    conDebitPeriodBeginBalance: aItem.conDebitPeriodBeginBalance,
                                    conDebitPeriodEndBalance: aItem.conDebitPeriodEndBalance,
                                    conDebitYearBeginBalance: aItem.conDebitYearBeginBalance,
                                    conYearCreditBalance: aItem.conYearCreditBalance,
                                    conYearDebitBalance: aItem.conYearDebitBalance,
                                    creditPeriodBeginBalance:
                                        type === 'allCurrency' && aItem.currencyZh === '折合本位币'
                                            ? aItem.conCreditPeriodBeginBalance
                                            : aItem.creditPeriodBeginBalance,
                                    debitPeriodBeginBalance:
                                        type === 'allCurrency' && aItem.currencyZh === '折合本位币'
                                            ? aItem.conDebitPeriodBeginBalance
                                            : aItem.debitPeriodBeginBalance,
                                    creditCurrPeriodBalance:
                                        type === 'allCurrency' && aItem.currencyZh === '折合本位币'
                                            ? aItem.conCreditCurrPeriodBalance
                                            : aItem.creditCurrPeriodBalance,
                                    debitCurrPeriodBalance:
                                        type === 'allCurrency' && aItem.currencyZh === '折合本位币'
                                            ? aItem.conDebitCurrPeriodBalance
                                            : aItem.debitCurrPeriodBalance,
                                    debitPeriodEndBalance:
                                        type === 'allCurrency' && aItem.currencyZh === '折合本位币'
                                            ? aItem.conDebitPeriodEndBalance
                                            : aItem.debitPeriodEndBalance,
                                    creditPeriodEndBalance:
                                        type === 'allCurrency' && aItem.currencyZh === '折合本位币'
                                            ? aItem.conCreditPeriodEndBalance
                                            : aItem.creditPeriodEndBalance,
                                    debitYearBeginBalance:
                                        type === 'allCurrency' && aItem.currencyZh === '折合本位币'
                                            ? aItem.conDebitYearBeginBalance
                                            : aItem.debitYearBeginBalance,
                                    creditYearBeginBalance:
                                        type === 'allCurrency' && aItem.currencyZh === '折合本位币'
                                            ? aItem.conCreditYearBeginBalance
                                            : aItem.creditYearBeginBalance,
                                    yearDebitBalance:
                                        type === 'allCurrency' && aItem.currencyZh === '折合本位币'
                                            ? aItem.conYearDebitBalance
                                            : aItem.yearDebitBalance,
                                    yearCreditBalance:
                                        type === 'allCurrency' && aItem.currencyZh === '折合本位币'
                                            ? aItem.conYearCreditBalance
                                            : aItem.yearCreditBalance,
                                })
                            })
                        }
                    } else {
                        treeLists.push({
                            ...this.getCommons(item),
                            children: this.resetTree(item.children, type),
                            ...this.setEmptyAmount(),
                        })
                    }
                })
                return treeLists
            },
            getCommons(item) {
                let obj = {
                    accountBooksId: item.accountBooksId,
                    isSubject: item.isSubject,
                    id: item.id,
                    parentId: item.parentId,
                    subjectCode: item.subjectCode,
                    subjectName: item.subjectName,
                    amountChild: [],
                }
                return obj
            },
            setMergeAmount(firstItem, secondItem) {
                let obj = {
                    currencyId: firstItem.currencyId,
                    currencyZh: firstItem.currencyZh,
                    conCreditCurrPeriodBalance: secondItem.conCreditCurrPeriodBalance,
                    conCreditPeriodBeginBalance: secondItem.conCreditPeriodBeginBalance,
                    conCreditPeriodEndBalance: secondItem.conCreditPeriodEndBalance,
                    conCreditYearBeginBalance: secondItem.conCreditYearBeginBalance,
                    conDebitCurrPeriodBalance: secondItem.conDebitCurrPeriodBalance,
                    conDebitPeriodBeginBalance: secondItem.conDebitPeriodBeginBalance,
                    conDebitPeriodEndBalance: secondItem.conDebitPeriodEndBalance,
                    conDebitYearBeginBalance: secondItem.conDebitYearBeginBalance,
                    conYearCreditBalance: secondItem.conYearCreditBalance,
                    conYearDebitBalance: secondItem.conYearDebitBalance,
                    creditCurrPeriodBalance: firstItem.creditCurrPeriodBalance,
                    creditPeriodBeginBalance: firstItem.creditPeriodBeginBalance,
                    creditPeriodEndBalance: firstItem.creditPeriodEndBalance,
                    creditYearBeginBalance: firstItem.creditYearBeginBalance,
                    debitCurrPeriodBalance: firstItem.debitCurrPeriodBalance,
                    debitPeriodBeginBalance: firstItem.debitPeriodBeginBalance,
                    debitPeriodEndBalance: firstItem.debitPeriodEndBalance,
                    debitYearBeginBalance: firstItem.debitYearBeginBalance,
                    yearCreditBalance: firstItem.yearCreditBalance,
                    yearDebitBalance: firstItem.yearDebitBalance,
                }
                return obj
            },
            setEmptyAmount() {
                let obj = {
                    currencyId: null,
                    currencyZh: null,
                    conCreditCurrPeriodBalance: 0,
                    conCreditPeriodBeginBalance: 0,
                    conCreditPeriodEndBalance: 0,
                    conCreditYearBeginBalance: 0,
                    conDebitCurrPeriodBalance: 0,
                    conDebitPeriodBeginBalance: 0,
                    conDebitPeriodEndBalance: 0,
                    conDebitYearBeginBalance: 0,
                    conYearCreditBalance: 0,
                    conYearDebitBalance: 0,
                    creditCurrPeriodBalance: 0,
                    creditPeriodBeginBalance: 0,
                    creditPeriodEndBalance: 0,
                    creditYearBeginBalance: 0,
                    debitCurrPeriodBalance: 0,
                    debitPeriodBeginBalance: 0,
                    debitPeriodEndBalance: 0,
                    debitYearBeginBalance: 0,
                    yearCreditBalance: 0,
                    yearDebitBalance: 0,
                }
                return obj
            },
            getSummaries(initTreeList) {
                let conCreditCurrPeriodBalance = initTreeList.map((v) => Number(v.conCreditCurrPeriodBalance)),
                    conCreditPeriodBeginBalance = initTreeList.map((v) => Number(v.conCreditPeriodBeginBalance)),
                    conCreditPeriodEndBalance = initTreeList.map((v) => Number(v.conCreditPeriodEndBalance)),
                    conCreditYearBeginBalance = initTreeList.map((v) => Number(v.conCreditYearBeginBalance)),
                    conDebitCurrPeriodBalance = initTreeList.map((v) => Number(v.conDebitCurrPeriodBalance)),
                    conDebitPeriodBeginBalance = initTreeList.map((v) => Number(v.conDebitPeriodBeginBalance)),
                    conDebitPeriodEndBalance = initTreeList.map((v) => Number(v.conDebitPeriodEndBalance)),
                    conDebitYearBeginBalance = initTreeList.map((v) => Number(v.conDebitYearBeginBalance)),
                    conYearCreditBalance = initTreeList.map((v) => Number(v.conYearCreditBalance)),
                    conYearDebitBalance = initTreeList.map((v) => Number(v.conYearDebitBalance)),
                    creditCurrPeriodBalance = initTreeList.map((v) => Number(v.creditCurrPeriodBalance)),
                    creditPeriodBeginBalance = initTreeList.map((v) => Number(v.creditPeriodBeginBalance)),
                    creditPeriodEndBalance = initTreeList.map((v) => Number(v.creditPeriodEndBalance)),
                    creditYearBeginBalance = initTreeList.map((v) => Number(v.creditYearBeginBalance)),
                    debitCurrPeriodBalance = initTreeList.map((v) => Number(v.debitCurrPeriodBalance)),
                    debitPeriodBeginBalance = initTreeList.map((v) => Number(v.debitPeriodBeginBalance)),
                    debitPeriodEndBalance = initTreeList.map((v) => Number(v.debitPeriodEndBalance)),
                    debitYearBeginBalance = initTreeList.map((v) => Number(v.debitYearBeginBalance)),
                    yearCreditBalance = initTreeList.map((v) => Number(v.yearCreditBalance)),
                    yearDebitBalance = initTreeList.map((v) => Number(v.yearDebitBalance))
                let obj = {
                    subjectCode: '合计',
                    conCreditCurrPeriodBalance: sum(conCreditCurrPeriodBalance),
                    conCreditPeriodBeginBalance: sum(conCreditPeriodBeginBalance),
                    conCreditPeriodEndBalance: sum(conCreditPeriodEndBalance),
                    conCreditYearBeginBalance: sum(conCreditYearBeginBalance),
                    conDebitCurrPeriodBalance: sum(conDebitCurrPeriodBalance),
                    conDebitPeriodBeginBalance: sum(conDebitPeriodBeginBalance),
                    conDebitPeriodEndBalance: sum(conDebitPeriodEndBalance),
                    conDebitYearBeginBalance: sum(conDebitYearBeginBalance),
                    conYearCreditBalance: sum(conYearCreditBalance),
                    conYearDebitBalance: sum(conYearDebitBalance),
                    creditCurrPeriodBalance: sum(creditCurrPeriodBalance),
                    creditPeriodBeginBalance: sum(creditPeriodBeginBalance),
                    creditPeriodEndBalance: sum(creditPeriodEndBalance),
                    creditYearBeginBalance: sum(creditYearBeginBalance),
                    debitCurrPeriodBalance: sum(debitCurrPeriodBalance),
                    debitPeriodBeginBalance: sum(debitPeriodBeginBalance),
                    debitPeriodEndBalance: sum(debitPeriodEndBalance),
                    debitYearBeginBalance: sum(debitYearBeginBalance),
                    yearCreditBalance: sum(yearCreditBalance),
                    yearDebitBalance: sum(yearDebitBalance),
                }
                let objTemp = {}
                for (const key in obj) {
                    objTemp[key] = obj[key]
                }
                this.sum.push(objTemp)
                this.treeLists.push(obj)
            },

            formatterAmount(value) {
                return value ? parseFormatNum(Number(value), 2) : ''
            },
            deleteEmptyParams() {
                let obj = {}
                for (const key in this.searchForm) {
                    if (this.searchForm[key] !== '') {
                        obj[key] = this.searchForm[key]
                        if (obj['queryTypes'] && obj['queryTypes'].length === 0) {
                            delete obj['queryTypes']
                        }
                    }
                }
                return obj
            },
            resetMethod() {
                this.searchForm = {
                    accountBooksId: '',
                    startPeriodId: '',
                    endPeriodId: '',
                    startSubjectCode: '',
                    endSubjectCode: '',
                    subjectLever: 4,
                    currencyId: this.currencyList.length ? this.currencyList[0].id : '',
                    queryTypes: '',
                }
                this.vouDurList.forEach((item) => {
                    if (item.isCurrentPeriod === 1) {
                        this.$set(this.searchForm, 'startPeriodId', item.periodId)
                        this.$set(this.searchForm, 'endPeriodId', item.periodId)
                    }
                })
                this.getList('search')
            },

            async getView(perms) {
                let { data } = await findView({ perms })
                if (data.ret === 1) {
                    let _data = data.data[0] || [],
                        _searches = _data.searches || [],
                        _buttons = _data.buttons || [],
                        _tables = _data.tables || []
                    this.searches = _searches.map((v) => v.value)
                    this.buttons = _buttons.map((v) => v.name)
                    this.tables = _tables
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
            exportFile() {
                let obj = this.deleteEmptyParams()
                let data = {
                    searchData: { ...obj },
                    excleName: this.$route.meta.title,
                    url: '/api/1/fmis/statSubjectBalanceController/find-all',
                    exportExcelsMethods: exportExcels,
                    dataNumOrTitle: this.$refs.umyTable.getCustomExcels(),
                    sum: this.sum,
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
                this.height = allH - searchH - 70
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
</style>
