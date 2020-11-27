<template>
    <div class="content-box">
        <!-- <h3>
            查看凭证
        </h3> -->
        <div :class="showSearch ? 'content wrap' : 'content'" ref="searchBox">
            <el-form :model="searchForm" inline>
                <el-row>
                    <el-form-item>
                        <el-select
                            v-model.number="accountBooksId"
                            placeholder="请选择帐套"
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
                    <div class="vouDur">
                        <el-form-item>
                            <!-- multiple :multiple-limit="2" -->
                            <el-select v-model="periodPrev" placeholder="期数查询范围" class="filter-item" size="mini">
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
                            <el-select v-model="periodNext" placeholder="期数查询范围" class="filter-item" size="mini">
                                <el-option
                                    v-for="item in vouDurList"
                                    :key="item.periodId"
                                    :label="item.fullValue"
                                    :value="item.periodId"
                                ></el-option>
                            </el-select>
                        </el-form-item>
                    </div>
                    <el-form-item>
                        <el-select
                            v-model="searchForm.status"
                            placeholder="选择凭证状态"
                            class="filter-item"
                            size="mini"
                            filterable
                            clearable
                        >
                            <el-option
                                v-for="item in statusList"
                                :key="item.value"
                                :label="item.label"
                                :value="item.value"
                            ></el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item>
                        <el-input
                            v-model.trim="searchForm.voucherNum"
                            placeholder="凭证号范围1-3"
                            size="mini"
                            clearable
                        ></el-input>
                    </el-form-item>
                    <el-form-item>
                        <el-input
                            v-model.trim="searchForm.briefValue"
                            placeholder="搜索摘要"
                            size="mini"
                            clearable
                        ></el-input>
                    </el-form-item>
                    <el-form-item>
                        <el-input
                            v-model.trim="searchForm.subjectName"
                            placeholder="搜索科目"
                            size="mini"
                            clearable
                        ></el-input>
                    </el-form-item>
                </el-row>
                <div class="btn-list">
                    <el-button @click="getList('search')" type="primary" size="mini" w-90>查询</el-button>
                    <el-button @click="resetMethod" size="mini" w-90>重置</el-button>
                    <i
                        :class="['iconfont icon-pulldown', showSearch ? 'pullup' : '']"
                        @click="showSearch = !showSearch"
                        v-if="showClose"
                    ></i>
                </div>
            </el-form>
        </div>
        <div class="table-box" ref="tableBox">
            <div class="table-btn">
                <el-button type="primary" @click="editOne('add')" w-90 size="mini">新增</el-button>
                <el-button
                    type="primary"
                    w-90
                    size="mini"
                    :disabled="selList.length === 0"
                    @click="checkVoucher('check')"
                    >审核</el-button
                >
                <el-button
                    type="primary"
                    w-90
                    size="mini"
                    :disabled="selList.length === 0"
                    @click="checkVoucher('uncheck')"
                    >反审核</el-button
                >
                <el-button @click="deleteOne('some')" :disabled="selList.length === 0" size="mini" w-90
                    >批量删除</el-button
                >
                <el-upload
                    class="upload-demo"
                    action="#"
                    :http-request="importFile"
                    :multiple="true"
                    :show-file-list="false"
                    style="display:inline-block"
                >
                    <el-button size="mini" w-90>导入</el-button>
                </el-upload>
                <el-button @click="exportFile()" w-90 size="mini" :disabled="tableData.list.length === 0"
                    >导出</el-button
                >
                <!-- <el-button @ w-90 size="mini" disabled>导出</el-button>
                <el-button size="mini" @click="reload()" class="fr" w-110>重置列宽</el-button> -->
                <el-button size="mini" w-110 icon="el-icon-s-operation" class="fr" @click="opencusColDialog"
                    >自定义列</el-button
                >
            </div>
            <div class="table-content">
                <dialogCustomTable
                    :tableData="tableData"
                    :textAlignRightList="textAlignRightList"
                    :pageNumber="pageNumber"
                    :pageSize="pageSize"
                    :customColumnDialog="customColumnDialog"
                    :tableHeaderHeight="'calc(100% - 97px)'"
                    :excelList="excelList"
                    :SpanMethod="SpanMethod"
                    @go-page="goPage"
                    @change-size="changeSize"
                    @selection-change="selectChange"
                    @handle-report="handleReport"
                    @getCheck-columnList="getCheckColumnList"
                >
                    <template slot="normal" slot-scope="scope">
                        <template v-if="scope.row.prop === 'trsamtd' || scope.row.prop === 'trsamtc'">
                            {{ scope.row.data[scope.row.prop] | formatAmount }}
                        </template>
                        <template v-else-if="scope.row.prop === 'voucherNumber'">
                            <el-link type="primary" :underline="false" @click="editOne('edit', scope.row.data)">{{
                                scope.row.data.voucherNumber
                            }}</el-link>
                        </template>
                        <template v-else>
                            {{ scope.row.data[scope.row.prop] }}
                        </template>
                    </template>
                    <template slot="oprate" slot-scope="scope">
                        <el-button small="small" type="text" @click="editOne('edit', scope.row)">编辑</el-button>
                        <el-button small="small" type="text" @click="deleteOne('one', scope.row)">删除</el-button>
                        <!-- <el-button small="small" type="text" @click="saveScroll">滚动高度</el-button> -->
                    </template>
                </dialogCustomTable>

                <!-- <el-table
                    :data="tableDataList"
                    :span-method="SpanMethod"
                    @selection-change="selectChange"
                    border
                    stripe
                    header-cell-class-name="tableHeader"
                    size="mini"
                    height="calc(100% - 94px)"
                    :cell-style="changeCellStyle"
                >
                    <el-table-column type="selection"></el-table-column>
                    <el-table-column prop="businessDate" label="日期" width="90"> </el-table-column>
                    <el-table-column prop="voucherNumber" label="凭证字号" width="80">
                        <template slot-scope="scope">
                            <el-link type="primary" :underline="false" @click="editOne('edit', scope.row)">{{
                                scope.row.voucherNumber
                            }}</el-link>
                        </template>
                    </el-table-column>
                    <el-table-column prop="brief" label="摘要" width="260"> </el-table-column>
                    <el-table-column prop="accountSubject" label="科目" width="420"> </el-table-column>
                    <el-table-column prop="trsamtd" label="借方金额" width="150">
                        <template slot-scope="scope">
                            <p>{{ scope.row.trsamtd | formatAmount }}</p>
                        </template>
                    </el-table-column>
                    <el-table-column prop="trsamtc" label="贷方金额" width="150">
                        <template slot-scope="scope">
                            <p>{{ scope.row.trsamtc | formatAmount }}</p>
                        </template>
                    </el-table-column>
                    <el-table-column prop="createZh" label="制单人" width="90"> </el-table-column>
                    <el-table-column prop="checkZh" label="审核人" width="90"> </el-table-column>
                    <el-table-column label="操作" width="130">
                        <template slot-scope="scope">
                            <el-button small="small" type="text" @click="editOne('edit', scope.row)">编辑</el-button>
                            <el-button small="small" type="text" @click="deleteOne('one', scope.row)">删除</el-button>
                        </template>
                    </el-table-column>
                </el-table>
                <Pagination
                    v-if="showPage"
                    :pageNumber="pageNumber"
                    :pageSize="pageSize"
                    :totalCount="tableData.totalCount"
                    :totalPage="tableData.totalPage"
                    @go-page="goPage"
                    @change-size="changeSize"
                ></Pagination> -->
            </div>
        </div>
    </div>
</template>
<script>
import { findAllAccountBook } from '@/api/fims/setSubject'
import { deleteBy, findAllVou, findVouDur, updateVoucher, importFlow } from '@/api/fims/voucherSelect'
// import Pagination from '@/components/pagination'
// import Bus from '@/utils/eventBus'
import { unique, removeEle, parseFormatNum } from '@/utils/index'
import dialogCustomTable from '@/components/dialog/dialogCustomTable'
import exportExcel from '@/utils/exportExcel.js'

export default {
    inject: ['reload'],
    name: 'examine',
    data() {
        return {
            searchForm: {},
            tableData: {
                list: [],
            },
            tableDataList: [],
            initTableList: [],
            spanArray: [],
            selList: [],
            pageSize: 100,
            pageNumber: 1,
            checkedIds: '',
            uncheckedIds: '',
            vouDurList: [],
            periodPrev: '',
            periodNext: '',
            accountBookList: [],
            accountBooksId: 1,
            goBackAccountBook: '',
            statusList: [
                {
                    value: 1,
                    label: '暂存',
                },
                {
                    value: 2,
                    label: '保存',
                },
                {
                    value: 3,
                    label: '已审核',
                },
                {
                    value: 11,
                    label: '未审核',
                },
                // {
                //     value: 4,
                //     label: '已过帐',
                // },
                {
                    value: 5,
                    label: '已结帐',
                },
            ],
            showSearch: false,
            showClose: false,
            contentDom: '',
            showPage: true,
            // 自定义列
            customColumnDialog: false,
            initEnter: false,
            textAlignRightList: ['贷方金额', '借方金额'],
            excelList: [],
        }
    },
    components: { dialogCustomTable },
    computed: {
        periodIds() {
            return [this.periodPrev, this.periodNext]
        },
        changeIsInitEnter() {
            return this.$store.state.isInitEnter
        },
        setVoucherTempData() {
            return this.$store.state.voucherTempData
        },
        changeIsNewlyIncreaseEnter() {
            return this.$store.state.isNewlyIncreaseEnter
        },
    },
    filters: {
        formatAmount(value) {
            let number = value * 1
            if (value === undefined || value === null || value === '' || value === 0) {
                return ''
            } else {
                return parseFormatNum(number, 2)
            }
        },
    },
    created() {
        // console.log('初次进入...')
        this.getVouDur()
        if (this.changeIsNewlyIncreaseEnter) {
            // console.log(this.setVoucherTempData)
            let {
                accountBooksId,
                periodPrev,
                periodNext,
                searchForm,
                pageSize,
                pageNumber,
                tTop,
                tLeft,
            } = this.setVoucherTempData
            this.$nextTick(() => {
                this.accountBooksId = accountBooksId
                this.periodPrev = periodPrev
                this.periodNext = periodNext
                this.searchForm = searchForm
                this.pageSize = pageSize
                this.pageNumber = pageNumber
                this.backTop = tTop
                this.backLeft = tLeft
                this.getAllAccountBook(accountBooksId)
            })
        } else {
            this.getAllAccountBook()
            // this.getList()
        }
    },
    mounted() {
        this.initEnter = true
        this.getInitHeight()
    },
    activated() {
        // console.log(this.setVoucherTempData)
        this.getList()
        if (this.initEnter) {
            this.$store.commit('GO_SCROLL_POSITION', this.$route)
        }
    },
    methods: {
        getInitHeight() {
            window.addEventListener('resize', this.$calcTableHeight())
            this.contentDom = this.$refs.searchBox
            this.showDetailBtnFun()
            window.addEventListener('resize', this.showDetailBtnFun)
        },
        saveScroll() {
            let tContainer = document.getElementsByClassName('el-table__body-wrapper')[0]
            let tTop = tContainer.scrollTop
            this.tTop = tTop
            let tLeft = tContainer.scrollLeft
            this.tLeft = tLeft
        },
        resetScroll() {
            let tContainer = this.$el.querySelector('.el-table__body-wrapper')
            tContainer.scrollTop = 0
            tContainer.scrollLeft = 0
        },

        showDetailBtnFun() {
            this.$nextTick(() => {
                this.showSearch = false
                if (50 < this.contentDom.scrollHeight) {
                    this.showClose = true
                } else {
                    this.showClose = false
                }
            })
        },
        async getAllAccountBook(accountBooksId) {
            let { data } = await findAllAccountBook({
                pageNo: 1,
                pageSize: 200,
            })
            if (data.ret === 1) {
                this.accountBookList = data.data.list
                if (this.changeIsNewlyIncreaseEnter) {
                    // 返回按钮回来
                    this.accountBooksId = accountBooksId
                    this.getList()
                    this.$store.commit('changeIsNewlyIncreaseEnter', false)
                } else {
                    this.accountBooksId = this.accountBookList[0].accountBooksId
                }
            } else {
                this.$message.error(data.msg)
            }
        },

        // 会计期间
        async getVouDur() {
            let { data } = await findVouDur({
                pageNo: 1,
                pageSize: 100,
            })
            if (data.ret === 1) {
                // console.log(data)
                this.vouDurList = data.data.list
            }
        },

        async getList(enterType) {
            if (enterType === 'search') {
                this.pageNumber = 1
                this.resetScroll()
            }
            if (enterType === 'init') {
                this.resetScroll()
            }
            if (this.searchForm.voucherNum) {
                let voucherNum = this.searchForm.voucherNum
                if (voucherNum.indexOf('-') === -1) {
                    let merge = `${voucherNum}-${voucherNum}`
                    this.$set(this.searchForm, 'voucherNum', merge)
                }
            }
            let postParmas = {
                pageNo: this.pageNumber,
                pageSize: this.pageSize,
                accountBooksId: this.accountBooksId,
                ...this.searchForm,
            }
            let { data } =
                this.periodPrev !== '' && this.periodNext !== ''
                    ? await findAllVou({
                          ...postParmas,
                          periodIds: this.periodIds,
                      })
                    : await findAllVou({
                          ...postParmas,
                      })
            if (data.ret === 1) {
                this.tableData = data.data
                let tableDataList = this.tableData.list
                this.initTableList = tableDataList
                // console.log(this.initTableList)
                // 修改数据结构，定义合并起始行
                let arr = []
                let s = 0
                let getTable = []
                tableDataList.forEach((item, i, data) => {
                    if (arr.length) {
                        s = arr[arr.length - 1].row + data[i - 1].entryInfo.length
                    }
                    arr.push({
                        row: s,
                        index: item.entryInfo.length,
                    })
                    if (item.entryInfo && item.entryInfo.length) {
                        item.entryInfo.forEach(subItem => {
                            getTable.push({
                                voucherId: item.voucherId,
                                businessDate: item.businessDate,
                                voucherNumber: item.voucherNumber,
                                brief: subItem.brief,
                                accountSubject: subItem.accountSubject,
                                trsamtd: subItem.trsamtd,
                                trsamtc: subItem.trsamtc,
                                createZh: item.createZh,
                                checkZh: item.checkZh,
                                status: item.status,
                            })
                        })
                    }
                })
                this.spanArray = arr
                this.tableDataList = getTable
                this.tableData.list = this.tableDataList.concat()

                // console.log(this.tableData)
                // console.log(this.tableDataList)
                // this.showPage = false
                this.$nextTick(() => {
                    // this.showPage = true
                    // if (!enterType) {  // 注释不要删除哈，这个页面的滚动条代码有点混乱
                    //     console.log(this.backTop, this.backTop)
                    //     let tContainer = this.$el.querySelector('.el-table__body-wrapper')
                    //     tContainer.scrollTop = this.backTop
                    //     tContainer.scrollLeft = this.backLeft
                    // }
                    // console.log(this.backTop, this.backTop)
                    this.$store.commit('changeIsInitEnter', true)
                    this.$store.state.voucherTempData = {}
                    // console.log('vuex查看凭证临时存储数据：', this.setVoucherTempData)
                })
            } else {
                this.$message.error(data.msg)
            }
        },
        exportFile() {
            this.$confirm(`即将导出 ${this.tableData.totalCount} 条数据, 是否继续?`, '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning',
            })
                .then(() => {
                    this.sureExport()
                })
                .catch(() => {
                    this.$message.info('已取消导出')
                })
        },
        async sureExport() {
            if (this.searchForm.voucherNum) {
                let voucherNum = this.searchForm.voucherNum
                if (voucherNum.indexOf('-') === -1) {
                    let merge = `${voucherNum}-${voucherNum}`
                    this.$set(this.searchForm, 'voucherNum', merge)
                }
            }
            let postParmas = {
                pageNo: 1,
                pageSize: this.tableData.totalCount,
                accountBooksId: this.accountBooksId,
                ...this.searchForm,
            }
            let { data } =
                this.periodPrev !== '' && this.periodNext !== ''
                    ? await findAllVou({
                          ...postParmas,
                          periodIds: this.periodIds,
                      })
                    : await findAllVou({
                          ...postParmas,
                      })
            if (data.ret === 1) {
                this.excelList = data.data.list
                // 修改数据结构，定义合并起始行
                let arr = []
                let s = 0
                let getTable = []
                this.excelList.forEach((item, i, data) => {
                    if (arr.length) {
                        s = arr[arr.length - 1].row + data[i - 1].entryInfo.length
                    }
                    arr.push({
                        row: s,
                        index: item.entryInfo.length,
                    })
                    if (item.entryInfo && item.entryInfo.length) {
                        item.entryInfo.forEach(subItem => {
                            getTable.push({
                                voucherId: item.voucherId,
                                businessDate: item.businessDate,
                                voucherNumber: item.voucherNumber,
                                brief: subItem.brief,
                                accountSubject: subItem.accountSubject,
                                trsamtd: subItem.trsamtd,
                                trsamtc: subItem.trsamtc,
                                createZh: item.createZh,
                                checkZh: item.checkZh,
                                status: item.status,
                            })
                        })
                    }
                })
                this.spanArray = arr
                this.excelList = getTable
                this.$nextTick(() => {
                    let name = this.$route.meta.title
                    exportExcel.exportExcelByDom('#excel-table0', name)
                })
            } else {
                this.$message.error(data.msg)
            }
        },
        async importFile(e) {
            let file = e.file
            let fileType = e.file.name.substring(e.file.name.lastIndexOf('.'))
            console.log(file, fileType)
            if (fileType.toLowerCase() !== '.xlsx' && fileType.toLowerCase() !== '.xls') {
                this.$message.warning('只支持上传xlsx格式文件')
                return false
            }

            let { data } = await importFlow({
                files: file,
            })
            if (data.ret === 1) {
                console.log('期初文件上传')
                this.$message.success(data.msg)
                this.getList()
            } else {
                this.$message.error(data.msg)
            }
        },
        // 合并表格行
        SpanMethod(data) {
            let { rowIndex, column } = data
            if (
                !column.property ||
                column.property === 'businessDate' ||
                column.property === 'voucherNumber' ||
                column.property === 'createZh' ||
                column.property === 'checkZh'
            ) {
                let obj = [0, 0]
                this.spanArray.some(v => {
                    if (rowIndex === v.row) {
                        obj = [v.index, 1]
                    }
                })
                return obj
            }
        },

        editOne(type, row) {
            // console.log(row, this.initTableList)  编辑接口 voucherId
            this.$store.commit('changeIsInitEnter', false)
            let checkedRow = []
            let parmas = {}
            let checkParmas = {}
            if (type === 'add') {
                parmas = {
                    type: 'add',
                    addStatus: true,
                    backStatus: true,
                    accountBooksId: this.accountBooksId,
                    periodPrev: this.periodPrev,
                    periodNext: this.periodNext,
                    pageSize: this.pageSize,
                    pageNumber: this.pageNumber,
                    searchForm: { ...this.searchForm },
                    tTop: 0,
                    tLeft: 0,
                }
                this.parmas = parmas
                this.$store.commit('setVoucherTempData', parmas)
            } else {
                this.saveScroll()
                checkedRow = this.initTableList.filter(v => {
                    if (v.voucherId === row.voucherId) {
                        return v
                    }
                })
                if (checkedRow[0].status === 2) {
                    checkParmas = {
                        checkStatus: true,
                        deleteStatus: true,
                    }
                } else if (checkedRow[0].status === 3) {
                    checkParmas = {
                        uncheckStatus: true,
                        isEdit: true,
                    }
                }
                parmas = {
                    type: 'edit',
                    addStatus: false,
                    backStatus: true,
                    checkedVou: checkedRow,
                    periodPrev: this.periodPrev,
                    periodNext: this.periodNext,
                    accountBooksId: this.accountBooksId,
                    pageSize: this.pageSize,
                    pageNumber: this.pageNumber,
                    searchForm: { ...this.searchForm },
                    tTop: this.tTop,
                    tLeft: this.tLeft,
                }
                // console.log(checkedRow)
                this.parmas = checkedRow[0].status === 1 ? parmas : Object.assign(checkParmas, parmas)
                // console.log(this.parmas)
                this.$store.commit('setVoucherTempData', this.parmas)
            }
            this.$router.push('/fims/voucher/newly')
        },

        async deleteProject(type, row) {
            // console.log(type, row)
            let { data } =
                type === 'one'
                    ? await deleteBy({
                          voucherIds: [row.voucherId],
                      })
                    : await deleteBy({
                          voucherIds: this.selList,
                      })
            if (data.ret === 1) {
                this.$message.success('删除成功!')
                this.getList()
            } else {
                this.$message.error(data.msg)
            }
        },
        deleteOne(type, row) {
            let text = ''
            text =
                type === 'one'
                    ? `确定删除日期为 ${row.businessDate}，凭证号为 ${row.voucherNumber} 的数据吗？`
                    : `当前选中 ${this.selList.length} 条数据，确定删除吗?`
            this.$confirm(text, '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning',
            })
                .then(() => {
                    this.deleteProject(type, row)
                })
                .catch(() => {
                    this.$message.info('已取消删除')
                })
        },
        selectChange(rows) {
            let selList = rows.map(item => item.voucherId)
            this.selList = unique(selList)
            // console.log(this.selList)
            let checkedIds = rows.map(item => {
                if (item.status === 2) {
                    return item.voucherId
                }
            })
            this.checkedLen = removeEle(unique(checkedIds), undefined).length
            this.checkedIds = removeEle(unique(checkedIds), undefined).join(',')
            // console.log(this.checkedIds, this.checkedLen, this.initTableList.length)

            let uncheckIds = rows.map(item => {
                if (item.status === 3) {
                    return item.voucherId
                }
            })
            this.uncheckLen = removeEle(unique(uncheckIds), undefined).length
            this.uncheckIds = removeEle(unique(uncheckIds), undefined).join(',')
            // console.log(this.uncheckIds, this.uncheckLen)
        },

        checkVoucher(type) {
            let voucherIds = ''
            let len = 0
            let skip = 0
            let text = ''
            let hint = ''
            if (type === 'check') {
                voucherIds = this.checkedIds
                len = this.checkedLen
                skip = this.selList.length - len
                text = '审核'
                hint = '（暂存/审核/已经结账的凭证，将会被跳过！）'
            } else {
                voucherIds = this.uncheckIds
                len = this.uncheckLen
                skip = this.selList.length - len
                text = '反审核'
                hint = '（没有审核的凭证或者已经结账的凭证，将会被跳过！）'
            }
            this.$confirm(`即将${text}的数据：${len}，跳过：${skip}，${hint}`, '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning',
            })
                .then(() => {
                    if (len < 1) {
                        this.$message({
                            type: 'error',
                            message: '操作无效，请选中至少一条符合条件的凭证，再进行确认！',
                        })
                    } else {
                        this.postCheck(type, voucherIds)
                    }
                })
                .catch(() => {
                    this.$message({
                        type: 'info',
                        message: `已取消${text}`,
                    })
                })
        },

        async postCheck(type, voucherIds) {
            let { data } = await updateVoucher({
                optionType: type,
                voucherIds: voucherIds,
            })
            if (data.ret === 1) {
                this.$message.success(data.msg)
                this.getList()
            } else {
                this.$message.error(data.msg)
            }
        },

        changeSize(pageSize) {
            this.pageSize = pageSize
            this.pageNum = 1
            this.getList('init')
        },
        goPage(pageNum) {
            this.pageNumber = pageNum
            this.getList('init')
        },
        cancelMethod() {
            this.dialogVisible = false
            this.form = {}
        },
        resetMethod() {
            this.searchForm = {}
            this.periodPrev = ''
            this.periodNext = ''
            this.pageNumber = 1
            this.getList('search')
        },
        // changeCellStyle(row) {
        //     if (row.column.label === '借方金额') {
        //         return 'textAlign: right'
        //     }
        //     if (row.column.label === '贷方金额') {
        //         return 'textAlign: right'
        //     }
        // },
        opencusColDialog() {
            this.customColumnDialog = true
        },
        handleReport() {
            this.customColumnDialog = false
        },
        getCheckColumnList() {
            this.getList()
        },
    },
}
</script>
<style lang="less" scoped>
@import url('../common.less');

.content-box {
    .content {
        .vouDur {
            display: inline-block;
            span {
                display: inline-block;
                line-height: 42px;
            }
        }
    }
    .table-box {
        .table-content {
            /deep/.el-table {
                .el-table__body-wrapper {
                    height: 652px;
                    overflow: auto;
                }
            }
        }
    }
}
</style>
