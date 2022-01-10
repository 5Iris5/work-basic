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
                            clearable
                        >
                            <el-option
                                v-for="(item, index) in accountBookList"
                                :key="index"
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
                                filterable
                                clearable
                            >
                                <el-option
                                    v-for="(item, index) in vouDurList"
                                    :key="index"
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
                                filterable
                                clearable
                            >
                                <el-option
                                    v-for="(item, index) in vouDurList"
                                    :key="index"
                                    :label="item.fullValue"
                                    :value="item.periodId"
                                ></el-option>
                            </el-select>
                        </el-form-item>
                    </div>
                    <el-form-item v-show="searches.includes('projectStatus')">
                        <el-select
                            v-model.number="searchForm.projectStatus"
                            placeholder="是否指定项目"
                            class="filter-item"
                            size="mini"
                            filterable
                            clearable
                            :disabled="
                                (roles.includes(1) || roles.includes(4)) && (!roles.includes(2) || !roles.includes(3))
                            "
                        >
                            <el-option
                                v-for="(item, index) in chooseProjectList"
                                :key="index"
                                :label="item.label"
                                :value="item.value"
                            ></el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item class="vouDur" v-show="searches.includes('currencyId')">
                        <el-select
                            v-model.number="searchForm.currencyId"
                            placeholder="币别"
                            size="mini"
                            filterable
                            clearable
                        >
                            <el-option
                                v-for="(item, index) in currencyList"
                                :key="index"
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
                            filterable
                            clearable
                        >
                            <el-option
                                v-for="(item, index) in allProjectList"
                                :key="index"
                                :label="`${item.cashCode} ${item.assisTypeName}`"
                                :value="item.id"
                            ></el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item v-show="searches.includes('userId')">
                        <el-select
                            v-model.number="searchForm.userId"
                            placeholder="项目审批人"
                            class="filter-item"
                            size="mini"
                            filterable
                            clearable
                            :disabled="
                                (roles.includes(1) || roles.includes(4)) && (!roles.includes(2) || !roles.includes(3))
                            "
                        >
                            <el-option
                                v-for="(item, index) in employerList"
                                :key="index"
                                :label="`${item.userNo} ${item.userName}`"
                                :value="item.userId"
                            ></el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item v-show="searches.includes('departId')">
                        <el-select
                            v-model="searchForm.departId"
                            placeholder="部门"
                            class="filter-item"
                            size="mini"
                            clearable
                            filterable
                        >
                            <el-option
                                v-for="(item, index) in departList"
                                :key="index"
                                :value="item.id"
                                :label="item.assisTypeName"
                            ></el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item v-show="searches.includes('departCheckUserId')">
                        <el-select
                            v-model.number="searchForm.departCheckUserId"
                            placeholder="部门审批人"
                            class="filter-item"
                            size="mini"
                            filterable
                            clearable
                        >
                            <el-option
                                v-for="(item, index) in departEmployerList"
                                :key="index"
                                :label="`${item.userNo} ${item.userName}`"
                                :value="item.userId"
                            ></el-option>
                        </el-select>
                    </el-form-item>
                </el-row>
                <div class="btn-list">
                    <el-button @click="getList('search')" type="primary" size="mini" w-90>查询</el-button>
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
                    v-show="buttons.includes('项目')"
                    :disabled="selList.length !== 1"
                    @click="editProject('one')"
                    type="primary"
                    size="mini"
                    >项目</el-button
                >
                <span
                    v-show="Boolean(columnList.length)"
                    v-for="(item, index) in projectAmountList"
                    :key="index"
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
                            <span class="specialcolor" @click="editOne('projectExamine', scope.row.data)">{{
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
                            <span class="specialcolor" @click="editProject('detail', scope.row.data)">
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
        <project
            :showProject="showProject"
            :projectList="projectList"
            :subProjectList="subProjectList"
            @cancelProjectDialog="cancelProjectDialog"
            @cofirmProjectDialog="cofirmProjectDialog"
            :isNormalShow="isNormalShow"
            :departList="departList"
            @focus-input="focusInput"
            @blur-input="blurInput"
            :isLock="isLock"
        ></project>
    </div>
</template>

<script>
    import uTable from '@/components/view/umyTable'
    import { findView } from '@/api/system/menu'
    import { findEmployeer, findRoles } from '@/api/fims/accountBook'
    import project from '@/components/voucher/projectSelect'
    import { findAllAccountBook } from '@/api/fims/setSubject'
    import { findAllCurrency } from '@/api/fims/setPeriod'
    import {
        findVouDur,
        designatedProject,
        prestoreProject,
        findAllVou,
        dropDownList,
        dropDownAll,
    } from '@/api/fims/voucherSelect'
    import { unique, parseFormatNum, deepCopy, delCommafy, resetColProps } from '@/utils/index'
    import { exportExcels, findAssis } from '@/api/fims/common'

    export default {
        name: 'projectExamine',
        components: { uTable, project },
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
                textAlignRightList: ['trsamtd', 'conTrsamtd', 'conTrsamtc', 'trsamtc'],
                spanFiled: ['businessDate', 'voucherNumber', 'companyName'],

                departEmployerList: [],
                departList: [],
                employerList: [],
                isNormalShow: true,
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
                // 搜索栏
                showSearch: false,
                showClose: false,
                contentDom: '',
                chooseProjectList: [
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
                startPeriodId: '',
                endPeriodId: '',
                projectKey: null,
                showProject: false,
                projectList: [],
                subProjectList: [],
                allProjectList: [],
                projectAmountList: [],
                projectItem: [],
                isLock: true, // 默认没有锁定可更改，弹窗确定按钮放开
                roles: [],
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
            this.formOtherEnter()
            this.initEnter ? (this.initEnter = false) : this.getList()
        },
        methods: {
            getInit() {
                this.getRole()
                let isAnalyseEnter = JSON.parse(sessionStorage.getItem('isAnalyseEnter'))
                this.initEnter && isAnalyseEnter ? this.getAllAccountBook(true) : this.getAllAccountBook()
                this.getSubProject()
                this.getAllProject()
                this.getAllDepart()
                this.getAllCurrency()
                this.getDepartEmployer()
            },
            formOtherEnter() {
                let isAnalyseEnter = JSON.parse(sessionStorage.getItem('isAnalyseEnter'))
                // console.log('activated: ', typeof isAnalyseEnter, isAnalyseEnter)
                if (isAnalyseEnter) {
                    this.pageNumber = 1
                    let parmas = deepCopy(JSON.parse(sessionStorage.getItem('analyseData')))
                    this.$set(this.searchForm, 'projectStatus', 2)
                    let {
                        itemId,
                        startPeriodId,
                        endPeriodId,
                        accountBooksId,
                        currencyId,
                        userId,
                        departId,
                        departCheckUserId,
                    } = parmas
                    this.$set(this.searchForm, 'itemId', itemId)
                    this.$set(this.searchForm, 'currencyId', currencyId)
                    this.$set(this.searchForm, 'userId', userId)
                    this.$set(this.searchForm, 'departCheckUserId', departCheckUserId)
                    this.$set(this.searchForm, 'departId', departId)
                    this.$set(this.searchForm, 'accountBooksId', accountBooksId)
                    this.startPeriodId = startPeriodId
                    this.endPeriodId = endPeriodId
                    sessionStorage.setItem('isAnalyseEnter', false)
                    sessionStorage.setItem('analyseData', JSON.stringify({}))
                }
            },
            async getAllAccountBook(bool) {
                let { data } = await findAllAccountBook({
                    pageNo: 1,
                    pageSize: 200,
                })
                if (data.ret === 1) {
                    this.accountBookList = data.data.list
                    if (!bool) this.$set(this.searchForm, 'accountBooksId', this.accountBookList[0].accountBooksId)
                    this.getVouDur(this.searchForm.accountBooksId, bool)
                } else {
                    this.$message.error(data.msg)
                }
            },
            changeAccountBook(val) {
                this.getVouDur(val)
            },
            async getVouDur(accountBooksId, bool) {
                let { data } = await findVouDur({
                    pageNo: 1,
                    pageSize: 100,
                    accountBooksId,
                })
                if (data.ret === 1) {
                    this.vouDurList = data.data.list
                    if (!bool) {
                        this.vouDurList.forEach((item) => {
                            if (item.isCurrentPeriod === 1) {
                                this.startPeriodId = this.endPeriodId = item.periodId
                            }
                        })
                    }
                } else {
                    this.$message.error(data.msg)
                }
                this.getList()
            },
            async getAllCurrency() {
                let { data } = await findAllCurrency({
                    pageNo: 1,
                    pageSize: 100,
                })
                data.ret === 1 ? (this.currencyList = data.data.list) : this.$message.error(data.msg)
            },
            async getRole() {
                let { data } = await findRoles({
                    pageNo: 1,
                    pageSize: 10,
                })
                if (data.ret) {
                    this.roles = data.data || []
                    this.getEmployer(this.roles)
                }
            },
            async getEmployer(roles) {
                let { data } = await findEmployeer({ busType: 1 })
                // if (this.employerList.length) this.employerList.push({ userId: -1, userNo: '', userName: '空' })
                if (data.ret === 1) {
                    this.employerList = data.data || []
                    if ((roles.includes(1) || roles.includes(4)) && (!roles.includes(2) || !roles.includes(3))) {
                        let _username = JSON.parse(localStorage.getItem('userInfo')).username
                        this.employerList.forEach((item) => {
                            if (item.userNameEn === _username) {
                                this.$set(this.searchForm, 'userId', item.userId)
                                this.$set(this.searchForm, 'projectStatus', 2)
                            }
                        })
                    }
                } else {
                    this.$message.error(data.msg)
                }
            },
            async getDepartEmployer() {
                let { data } = await findEmployeer({ busType: 2 })
                data.ret === 1 ? (this.departEmployerList = data.data || []) : this.$message.error(data.msg)
            },
            async getAllDepart() {
                let { data } = await findAssis({
                    pageNo: 1,
                    pageSize: 200,
                    assisTypeId: 2,
                })
                data.ret === 1 ? (this.departList = data.data.list) : this.$message.error(data.msg)
            },
            // 项目 - 所有
            async getAllProject() {
                let { data } = await dropDownAll({
                    assisTypeId: 10,
                })
                data.ret === 1 ? (this.allProjectList = data.data) : this.$message.error(data.msg)
            },
            // 项目 - 末级
            async getSubProject() {
                let { data } = await dropDownList({
                    assisTypeId: 10,
                })
                data.ret === 1 ? (this.subProjectList = data.data) : this.$message.error(data.msg)
            },

            async setPrestoreProject(saveProjectData) {
                let { data } = await prestoreProject({
                    ...saveProjectData,
                })
                if (data.ret === 1) {
                    this.$message.success(data.msg)
                    this.showProject = false
                    this.getList()
                } else {
                    this.$message.error(data.msg)
                }
            },
            async editProject(type, row) {
                let _projectkey = type === 'one' ? `${this.projectItem[0].voucherKey}` : `${row.voucherKey}`
                let { data } =
                    type === 'one'
                        ? await designatedProject({
                              voucherId: this.selList[0],
                          })
                        : await designatedProject({
                              voucherId: row.voucherId,
                          })
                if (data.ret === 1) {
                    let _data = data.data
                    this.isLock = _data.isLock
                    this.projectKey = `${_projectkey}_proj`
                    this.projectList = _data.sidvList.map((item) => {
                        item.statEntryinfo.trsamtc = item.statEntryinfo.trsamtc
                            ? this.formatterAmount(item.statEntryinfo.trsamtc)
                            : item.statEntryinfo.trsamtc
                        item.statEntryinfo.trsamtd = item.statEntryinfo.trsamtd
                            ? this.formatterAmount(item.statEntryinfo.trsamtd)
                            : item.statEntryinfo.trsamtd
                        item.statEntryinfo.conTrsamtc = item.statEntryinfo.conTrsamtc
                            ? this.formatterAmount(item.statEntryinfo.conTrsamtc)
                            : item.statEntryinfo.conTrsamtc
                        item.statEntryinfo.conTrsamtd = item.statEntryinfo.conTrsamtd
                            ? this.formatterAmount(item.statEntryinfo.conTrsamtd)
                            : item.statEntryinfo.conTrsamtd
                        return item
                    })
                    // console.log(this.projectList)
                    this.$nextTick(() => {
                        this.getSubProject()
                        this.showProject = true
                    })
                } else {
                    this.$message.error(data.msg)
                }
            },
            cancelProjectDialog() {
                this.showProject = false
            },
            cofirmProjectDialog() {
                this.projectList = this.projectList.map((item) => {
                    item.statEntryinfo.trsamtc = item.statEntryinfo.trsamtc
                        ? Number(delCommafy(item.statEntryinfo.trsamtc))
                        : item.statEntryinfo.trsamtc
                    item.statEntryinfo.trsamtd = item.statEntryinfo.trsamtd
                        ? Number(delCommafy(item.statEntryinfo.trsamtd))
                        : item.statEntryinfo.trsamtd
                    item.statEntryinfo.conTrsamtc = item.statEntryinfo.conTrsamtc
                        ? Number(delCommafy(item.statEntryinfo.conTrsamtc))
                        : item.statEntryinfo.conTrsamtc
                    item.statEntryinfo.conTrsamtd = item.statEntryinfo.conTrsamtd
                        ? Number(delCommafy(item.statEntryinfo.conTrsamtd))
                        : item.statEntryinfo.conTrsamtd
                    return item
                })
                let saveProjectData = {
                    sidvList: this.projectList,
                    projectKey: this.projectKey,
                    isSave: true,
                    isAdjust: true,
                }
                this.$nextTick(() => {
                    this.setPrestoreProject(saveProjectData)
                })
            },
            focusInput(data) {
                let { row, target } = data
                let { urrId } = row
                this.projectList = this.projectList.map((item) => {
                    if (item.urrId === urrId) {
                        item.statEntryinfo[target] = delCommafy(item.statEntryinfo[target])
                    }
                    return item
                })
            },
            blurInput(data) {
                let { row, target } = data
                let { urrId } = row
                this.projectList = this.projectList.map((item) => {
                    if (item.urrId === urrId) {
                        item.statEntryinfo[target] = this.formatterAmount(item.statEntryinfo[target])
                    }
                    return item
                })
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
                    isProject: 1,
                    ...this.searchForm,
                    orderParams: this.sortParams,
                }
                let { data } =
                    this.startPeriodId && this.endPeriodId
                        ? await findAllVou({ periodIds: this.periodIds, ..._params })
                        : await findAllVou({ ..._params })
                if (data.ret === 1) {
                    this.tableData = data.data
                    this.projectAmountList = data.data.obj || []
                    this.tableDataList =
                        data.data && data.data.list.length ? this.setSpanstyle(data.data.list, this.spanFiled) : []
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
            setSpanstyle(tableDataList, spanFiled) {
                let getTable = []
                tableDataList.forEach((item) => {
                    if (item.entryInfo && item.entryInfo.length)
                        item.entryInfo.forEach((subItem, subIdx) => {
                            let _info = {
                                voucherId: item.voucherId,
                                voucherNumber: item.voucherNumber,
                                businessDate: item.businessDate,
                                brief: subItem.brief,
                                urrId: subItem.urrId,
                                urrId2: subItem.urrId2,
                                accountSubject: subItem.accountSubject,
                                currencyId: subItem.currencyId,
                                trsamtd: subItem.trsamtd,
                                trsamtc: subItem.trsamtc,
                                conTrsamtd: subItem.conTrsamtd,
                                conTrsamtc: subItem.conTrsamtc,
                                cashAmount: subItem.cashAmount,
                                itemName: subItem.itemName,
                                remark: subItem.remark,
                                companyName: item.companyName,
                                departName: subItem.departName,
                                voucherKey: item.voucherKey,
                            }
                            if (subIdx) {
                                for (const key in _info) {
                                    if (spanFiled.includes(key)) _info[key] = ''
                                }
                            }
                            _info.hideSelection = subIdx ? true : false
                            getTable.push(_info)
                        })
                })
                return getTable
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
                this.projectItem = rows.map((item) => item)
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
                        isProject: 1,
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
