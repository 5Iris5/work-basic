<template>
    <div class="content-box" ref="contentBox">
        <!-- <h3>
            查看凭证
        </h3> -->
        <div :class="showSearch ? 'content wrap' : 'content'" ref="searchBox">
            <el-form :model="searchForm" inline>
                <el-row>
                    <el-form-item v-show="searches.includes('accountBooksId')">
                        <el-select
                            v-model.number="searchForm.accountBooksId"
                            placeholder="请选择帐套"
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
                    <div class="vouDur" v-show="searches.includes('periodIds')">
                        <el-form-item>
                            <el-select v-model="periodPrev" placeholder="会计期间" class="filter-item" size="mini">
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
                            <el-select v-model="periodNext" placeholder="会计期间" class="filter-item" size="mini">
                                <el-option
                                    v-for="item in vouDurList"
                                    :key="item.periodId"
                                    :label="item.fullValue"
                                    :value="item.periodId"
                                ></el-option>
                            </el-select>
                        </el-form-item>
                    </div>
                    <el-form-item v-show="searches.includes('status')">
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
                    <el-form-item v-show="searches.includes('voucherNum')">
                        <el-input
                            v-model.trim="searchForm.voucherNum"
                            placeholder="凭证号范围1-3"
                            size="mini"
                            clearable
                        ></el-input>
                    </el-form-item>
                    <el-form-item v-show="searches.includes('currencyId')">
                        <el-select v-model.number="searchForm.currencyId" placeholder="币别" size="mini" clearable>
                            <el-option
                                v-for="item in currencyList"
                                :key="item.id"
                                :label="item.nameZh"
                                :value="item.id"
                            ></el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item v-show="searches.includes('briefValue')">
                        <el-input
                            v-model.trim="searchForm.briefValue"
                            placeholder="搜索摘要"
                            size="mini"
                            clearable
                        ></el-input>
                    </el-form-item>
                    <el-form-item v-show="searches.includes('subjectName')">
                        <el-input
                            v-model.trim="searchForm.subjectName"
                            placeholder="搜索科目"
                            size="mini"
                            clearable
                        ></el-input>
                    </el-form-item>
                    <el-form-item v-show="searches.includes('createUserId')">
                        <el-select v-model="searchForm.createUserId" placeholder="制单人" size="mini" clearable>
                            <el-option
                                v-for="(value, key) in createrList"
                                :key="key"
                                :label="value"
                                :value="key"
                            ></el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item v-show="searches.includes('voucherType')">
                        <el-select v-model="searchForm.voucherType" placeholder="系统模块" size="mini" clearable>
                            <el-option
                                v-for="(value, key) in systemList"
                                :key="key"
                                :label="value"
                                :value="key"
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
                <el-button v-show="buttons.includes('新增')" type="primary" @click="editOne('add')" size="mini"
                    >新增</el-button
                >
                <el-button
                    v-show="buttons.includes('复制')"
                    type="primary"
                    @click="editOne('copy')"
                    size="mini"
                    :disabled="selList.length !== 1"
                    >复制</el-button
                >
                <el-button v-show="buttons.includes('打印')" type="primary" size="mini" @click="print()"
                    >打印</el-button
                >
                <el-button
                    v-show="buttons.includes('审核')"
                    type="primary"
                    size="mini"
                    :disabled="selList.length === 0"
                    @click="checkVoucher('check')"
                    >审核</el-button
                >
                <el-button
                    v-show="buttons.includes('反审核')"
                    type="primary"
                    size="mini"
                    :disabled="selList.length === 0"
                    @click="checkVoucher('uncheck')"
                    >反审核</el-button
                >
                <el-button
                    v-show="buttons.includes('补号')"
                    type="primary"
                    size="mini"
                    :disabled="isAllow"
                    @click="countVoucher"
                    >补号</el-button
                >
                <el-button
                    v-show="buttons.includes('冲销')"
                    type="primary"
                    size="mini"
                    :disabled="selList.length !== 1"
                    @click="editOne('writtenoff')"
                    >冲销</el-button
                >
                <el-button
                    v-show="buttons.includes('批量删除')"
                    @click="deleteOne('some')"
                    :disabled="selList.length === 0"
                    size="mini"
                    >批量删除</el-button
                >
                <el-button v-show="buttons.includes('导入')" size="mini" @click="importExamine">导入</el-button>
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
                <p v-if="Boolean(columnList.length)" class="period-link">{{ periodTxt }}</p>
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
                    :showOperate="true"
                    :operateFixed="'left'"
                    :operateWidth="90"
                    :pageNumber="pageNumber"
                    :pageSize="pageSize"
                    :showCustomColumn="showCustomColumn"
                    :viewId="viewId"
                    :tableId="tableId"
                    :showCustomSort="showCustomSort"
                    :flag="'StatVoucherinfoEntity'"
                    @go-page="goPage"
                    @change-size="changeSize"
                    @select="handleSelect"
                    @selection-change="selectChange"
                    @get-tableProps-by-customColumn="getTableProps"
                    @submit-sort="submitCustomSort"
                    @reset-tableProps-sort="resetPropSort"
                >
                    <template slot="normal" slot-scope="scope">
                        <template v-if="scope.row.prop === 'voucherNumber'">
                            <span class="specialcolor" @click="editOne('edit', scope.row.data)">{{
                                scope.row.data.voucherNumber
                            }}</span>
                        </template>
                        <template v-else-if="scope.row.prop === 'brief'">
                            <span v-if="scope.row.tit.typeCode === 2">{{ scope.row.data[scope.row.prop] }}</span>
                            <el-input
                                v-else
                                class="needTransparent"
                                size="mini"
                                v-model.trim="scope.row.data[scope.row.prop]"
                                @change="
                                    changeBrief(scope.row.data, scope.row.data.urrId, scope.row.data[scope.row.prop])
                                "
                            ></el-input>
                        </template>
                        <template v-else-if="textAlignRightList.includes(scope.row.prop)">
                            <span>{{ formatterAmount(scope.row.data[scope.row.prop]) }}</span>
                        </template>
                        <template v-else-if="scope.row.prop === 'currencyId'">
                            <span>{{ formatterCurrency(scope.row.data[scope.row.prop]) }}</span>
                        </template>
                        <template v-else-if="scope.row.prop === 'invosNum'">
                            <span
                                v-show="
                                    (scope.row.data.voucherType === 2 || scope.row.data.voucherType === 7) &&
                                    scope.row.data.invosNum
                                "
                                class="specialcolor"
                                @click="showAttach(scope.row.data)"
                            >
                                {{ scope.row.data.invosNum }}</span
                            >
                            <span
                                v-show="
                                    scope.row.data.voucherType !== 2 &&
                                    scope.row.data.voucherType !== 7 &&
                                    scope.row.data.fjNum
                                "
                                class="specialcolor"
                                @click="showAttach(scope.row.data, 'rowspan')"
                                >{{ scope.row.data.fjNum }}</span
                            >
                        </template>
                        <template v-else-if="scope.row.prop === 'voucherType'">
                            <span
                                v-if="scope.row.data.voucherType === 2 && scope.row.data.invosNum"
                                class="specialcolor"
                                @click="expenseDetail(scope.row.data)"
                            >
                                {{ scope.row.data.voucherTypeZh }}</span
                            >
                            <span v-else>{{ scope.row.data.voucherTypeZh }}</span>
                        </template>
                        <template v-else
                            ><span>{{ scope.row.data[scope.row.prop] }}</span></template
                        >
                    </template>
                    <template slot="oprate" slot-scope="scope">
                        <span
                            v-show="tableOperate.includes('编辑')"
                            class="specialcolor"
                            @click="editOne('edit', scope.row)"
                            >编辑</span
                        >
                        <span
                            v-show="tableOperate.includes('删除')"
                            class="specialcolor"
                            @click="deleteOne('one', scope.row)"
                            >删除</span
                        >
                    </template>
                </uTable>
                <div v-else class="no-roles">
                    <img src="../../../assets/images/role.png" alt="暂无此页面权限" />
                    <p>暂无此页面权限!</p>
                </div>
            </div>
        </div>
        <dialogImport
            :title="''"
            :importVisible="importVisible"
            :downloadName="downloadName"
            :downloadUrl="downloadUrl"
            @cancel-method="cancelImport"
            @confirm-method="confirmImport"
        ></dialogImport>
        <dialogAttach v-model="attachVisible" :tabList="tabList" :typeData="typeData"> </dialogAttach>
        <el-dialog
            ref="printDialog"
            v-moveallow
            :close-on-click-modal="false"
            title="打印设置"
            :visible.sync="printDialog"
            width="50%"
            @close="closePrintMethod"
            class="detail-dialog"
        >
            <printCondition ref="printCondition" :printVou="true"></printCondition>
            <div slot="footer" class="dialog-footer">
                <el-button type="primary" @click="savePrintMethod" size="mini" w-65>打 印</el-button>
            </div>
        </el-dialog>
    </div>
</template>
<script>
    import uTable from '@/components/view/umyTable'
    import dialogImport from '@/components/dialog/dialogImport'
    import dialogAttach from '@/components/dialog/dialogAttach'
    import printCondition from '@/components/voucher/print'
    import { findView } from '@/api/system/menu'
    import { findAllAccountBook } from '@/api/fims/setSubject'
    import {
        deleteBy,
        findAllVou,
        findVouDur,
        updateVoucher,
        importFlow,
        updateEntryBrief,
        setCountVou,
        findBillIdByUrrId,
        queryVOuCreater,
        queryVouType,
    } from '@/api/fims/voucherSelect'
    import { unique, removeEle, parseFormatNum, resetColProps } from '@/utils/index'
    import { findCurrency } from '@/api/fims/common'
    import { exportExcels } from '@/api/fims/common'

    export default {
        name: 'examine',
        components: { uTable, dialogImport, dialogAttach, printCondition },
        computed: {
            periodIds() {
                return [this.periodPrev, this.periodNext]
            },
            isAllow() {
                return !this.selList.length || !this.searchForm.accountBooksId || !this.periodPrev ? true : false
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
                textAlignRightList: ['originalAmount', 'conTrsamtd', 'conTrsamtc'],
                colFixedList: [],
                spanFiled: ['businessDate', 'voucherNumber', 'createZh', 'checkZh', 'invosNum', 'fjNum', 'voucherTypeZh'],

                printDialog: false,
                // 附件
                attachVisible: false,
                tabList: [],
                typeData: {},
                // 导入弹框
                importVisible: false,
                downloadUrl: `/files/公司简称-年-月-业务凭证导入模板.xlsx`,
                downloadName: '凭证模板',
                currencyList: [],
                createrList: {},
                systemList: {},
                searchForm: {},
                tableData: {
                    list: [],
                },
                tableDataList: [],
                initTableList: [],
                selList: [],
                pageSize: 100,
                pageNumber: 1,
                checkedIds: '',
                uncheckedIds: '',
                vouDurList: [],
                periodPrev: '',
                periodNext: '',
                accountBookList: [],
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
                initEnter: false,
                ckdVouIds: [], // 取消选中的selected
                periodTxt: null,
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
            document.addEventListener('keydown', this.handleKeyEvent)
        },
        methods: {
            getInit() {
                this.getAllCurrency()
                this.getCreaterList()
                this.getQueryVouType()
                this.getAllAccountBook()
            },
            async getAllCurrency() {
                let { data } = await findCurrency({
                    pageNo: 1,
                    pageSize: 10000,
                })
                if (data.ret === 1) {
                    this.currencyList = data.data.list
                } else {
                    this.$message.error(data.msg)
                }
            },
            async getCreaterList() {
                let { data } = await queryVOuCreater({})
                if (data.ret === 1) {
                    this.createrList = data.data
                } else {
                    this.$message.error(data.msg)
                }
            },
            async getQueryVouType() {
                let { data } = await queryVouType({})
                if (data.ret === 1) {
                    this.systemList = data.data
                } else {
                    this.$message.error(data.msg)
                }
            },
            changeAccountBook(accountBooksId) {
                this.getVouDur(accountBooksId, true)
            },
            async getAllAccountBook() {
                let { data } = await findAllAccountBook({
                    pageNo: 1,
                    pageSize: 100,
                })
                if (data.ret === 1) {
                    this.accountBookList = data.data.list
                    this.$set(this.searchForm, 'accountBooksId', this.accountBookList[0].accountBooksId)
                    this.getVouDur(this.searchForm.accountBooksId)
                } else {
                    this.$message.error(data.msg)
                }
            },
            async getVouDur(accountBooksId, bool) {
                let { data } = await findVouDur({
                    pageNo: 1,
                    pageSize: 100,
                    accountBooksId,
                })
                if (data.ret === 1) {
                    this.vouDurList = data.data.list
                    this.vouDurList.forEach((item) => {
                        if (item.isCurrentPeriod === 1) {
                            if (!bool) this.periodPrev = this.periodNext = item.periodId
                            this.periodTxt = `当前会计期间: ${item.year}年${item.period}月`
                        }
                    })
                } else {
                    this.$message.error(data.msg)
                }
                if (!bool) this.getList()
            },

            /**
             * 键盘事件
             *  1. shift+c 复制凭证，2. shift+v 冲销凭证
             */
            handleKeyEvent(e) {
                if (e.key.toUpperCase() === 'C' && e.shiftKey) {
                    e.preventDefault()
                    if (this.selList.length !== 1) return
                    this.editOne('copy')
                }
                if (e.key.toUpperCase() === 'V' && e.shiftKey) {
                    e.preventDefault()
                    if (this.selList.length !== 1) return
                    this.editOne('writtenoff')
                }
            },

            // 查看报销明细
            expenseDetail(row) {
                this.getDetailId(row.urrId)
            },
            // 明细id获取
            async getDetailId(urrId) {
                let { data } = await findBillIdByUrrId({
                    urrId,
                })
                if (data.ret === 1) {
                    if (!data.data.billIds[0]) this.$message.error('无对应报销明细')
                    else {
                        this.$router.push({
                            name: 'expenseRec',
                            params: { id: data.data.billIds[0] },
                        })
                    }
                } else {
                    this.$message.error(data.msg)
                }
            },
            editOne(type, row) {
                // console.log(row, this.initTableList)  编辑接口 voucherId
                sessionStorage.setItem('setVoucherTempData', JSON.stringify({}))
                sessionStorage.setItem('changeIsInitEnter', false)
                let parmas = {}
                if (type === 'add') {
                    parmas = {
                        type: 'add',
                        accountBooksId: this.searchForm.accountBooksId,
                    }
                    sessionStorage.setItem('setVoucherTempData', JSON.stringify(parmas))
                }
                if (type === 'edit') {
                    parmas = {
                        type: 'edit',
                        voucherId: row.voucherId,
                    }
                    sessionStorage.setItem('setVoucherTempData', JSON.stringify(parmas))
                }
                /**
                 * 复制凭证接口 ->
                 * 1. 根据当前帐套id，获取凭证号/会计期间
                 * 2. 根据凭证voucherId， 获取凭证数据
                 * 3. 删除凭证id,分录id,状态status修改为新增,voucherType修改为1手动新增的凭证
                 **/
                if (type === 'copy') {
                    parmas = {
                        type: 'copy',
                        // voucherType: 1,
                        voucherId: this.selList[0],
                    }
                    // this.$store.commit('setVoucherTempData', parmas)
                    sessionStorage.setItem('setVoucherTempData', JSON.stringify(parmas))
                }
                /**
                 * 冲销凭证接口 ->
                 * 1. 复用复制凭证逻辑
                 * 2. 金额为相反数
                 * 3. 第一行摘要：冲xxx期 xxx号凭证，如：冲2020年8期18号凭证，其他行摘要不变
                 */
                if (type === 'writtenoff') {
                    parmas = {
                        type: 'writtenoff',
                        voucherId: this.selList[0],
                    }
                    sessionStorage.setItem('setVoucherTempData', JSON.stringify(parmas))
                }
                this.$router.push('/fims/voucher/newly')
            },
            // 不分凭证状态修改摘要
            async changeBrief(row, urrId, brief) {
                // console.log(row, urrId, brief)
                let { data } = await updateEntryBrief({
                    urrId: urrId,
                    brief: brief,
                })
                if (data.ret === 1) {
                    this.$message.success(data.msg)
                    this.getList()
                } else {
                    this.$message.error(data.msg)
                }
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
                            this.$message.error({
                                message: '操作无效，请选中至少一条符合条件的凭证，再进行确认！',
                            })
                        } else {
                            this.postCheck(type, voucherIds)
                        }
                    })
                    .catch(() => {
                        this.$message.info({
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
            async countVoucher() {
                let { data } = await setCountVou({
                    accountBooksId: this.searchForm.accountBooksId,
                    periodId: Number(this.periodPrev),
                    voucherIds: this.selList,
                })
                if (data.ret === 1) {
                    this.$message.success(data.msg)
                    this.getList()
                } else {
                    this.$message.error(data.msg)
                }
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
            importExamine() {
                this.importVisible = true
            },
            cancelImport() {
                this.importVisible = false
            },
            async confirmImport(file) {
                let { data } = await importFlow({
                    files: file,
                })
                if (data.ret === 1) {
                    this.$message.success(data.msg)
                    this.importVisible = false
                    this.getList()
                } else {
                    this.$message.error(data.msg)
                }
            },

            showAttach(row, isRowspan) {
                this.attachVisible = true
                // OA单据
                if (row.voucherType === 4) {
                    // 付款单
                    this.tabList = ['其他', '发票']
                } else {
                    this.tabList = ['发票', '其他']
                }
                if (isRowspan) {
                    // 合并行附件，传凭证id
                    this.typeData = { voucherId: row.voucherId }
                } else {
                    // 非合并行附件，传分录urrId
                    this.typeData = { urrId: row.urrId }
                }
            },

            print() {
                this.printDialog = true
                let len = this.selList.length ? true : false
                this.$nextTick(() => {
                    this.$refs.printCondition.prange = len ? 2 : 1
                    this.$refs.printCondition.getAllCase()
                })
            },
            closePrintMethod() {
                this.printDialog = false
                this.$refs.printCondition.activeName = 'size'
                this.$refs.printCondition.prange = 1
            },
            savePrintMethod() {
                let { allowRight, printsetList } = this.$refs.printCondition.checkPrint()
                if (allowRight) {
                    let voucherIds = this.selList.join(),
                        prange = this.$refs.printCondition.prange
                    this.printVoucher(voucherIds, printsetList, prange)
                }
            },
            async printVoucher(voucherIds, printsetList, prange) {
                let postParmas = {
                    pageNo: this.pageNumber,
                    pageSize: this.pageSize,
                    ...this.searchForm,
                    voucherIds,
                    printSet: printsetList,
                    prange,
                    isprint: 1,
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
                    let _pdf = data.data
                    if (_pdf) window.open(_pdf, '_blank')
                } else {
                    this.$message.error(data.msg)
                }
            },

            async getList(search) {
                if (search && search !== 'GoPage') this.pageNumber = 1
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
                    ...this.searchForm,
                    orderParams: this.sortParams,
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
                                voucherTypeZh: item.voucherTypeZh,
                                voucherType: item.voucherType,
                                fjNum: item.fjNum,
                                invosNum: subItem.invosNum,
                                voucherId: item.voucherId,
                                businessDate: item.businessDate,
                                voucherNumber: item.voucherNumber,
                                brief: subItem.brief,
                                urrId: subItem.urrId,
                                accountSubject: subItem.accountSubject,
                                conTrsamtd: subItem.conTrsamtd,
                                conTrsamtc: subItem.conTrsamtc,
                                createZh: item.createZh,
                                checkZh: item.checkZh,
                                status: item.status,
                                rate: subItem.rate,
                                currencyId: subItem.currencyId,
                                trsamtc: subItem.trsamtc,
                                trsamtd: subItem.trsamtd,
                                originalAmount: subItem.trsamtc ? subItem.trsamtc : subItem.trsamtd,
                            }
                            if (subIdx) {
                                for (const key in _info) {
                                    if (spanFiled.includes(key)) {
                                        _info[key] = ''
                                        // 报销单|报销明细->附件数|系统模块不合并
                                        if (item.voucherType === 2 || item.voucherType === 7) {
                                            _info['voucherType'] = item.voucherType
                                            _info['voucherTypeZh'] = item.voucherTypeZh
                                            _info['fjNum'] = item.fjNum
                                            _info['invosNum'] = item.invosNum
                                        }
                                    }
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
                this.periodPrev = this.periodNext = this.periodTxt = ''
                this.$set(this.searchForm, 'accountBooksId', this.accountBookList[0].accountBooksId)
                this.getList('search')
            },
            selectChange(rows) {
                let selList = rows.map((item) => item.voucherId)
                this.selList = unique(selList)
                // 全选按钮后操作个别数据
                if (!this.selList.length) this.ckdVouIds = []
                this.selList = this.selList.filter((v) => this.ckdVouIds.indexOf(v) === -1)
                // console.log('凭证voucherId个数： ', this.selList.length)
                let checkedIds = rows.map((item) => {
                    if (item.status === 2) {
                        return item.voucherId
                    }
                })
                this.checkedLen = removeEle(unique(checkedIds), undefined).length
                this.checkedIds = removeEle(unique(checkedIds), undefined).join(',')
                // console.log(this.checkedIds, this.checkedLen, this.initTableList.length)

                let uncheckIds = rows.map((item) => {
                    if (item.status === 3) {
                        return item.voucherId
                    }
                })
                this.uncheckLen = removeEle(unique(uncheckIds), undefined).length
                this.uncheckIds = removeEle(unique(uncheckIds), undefined).join(',')
                // console.log(this.uncheckIds, this.uncheckLen)
            },
            handleSelect(selects) {
                let { current, selected } = selects
                if (!selected) {
                    this.ckdVouIds.push(current.voucherId)
                } else {
                    this.ckdVouIds.splice(this.ckdVouIds.indexOf(current.voucherId), 1)
                }
                // console.log('取消勾选总数据: ', selected, current.voucherId, this.ckdVouIds)
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
            document.removeEventListener('keydown', this.handleKeyEvent)
            window.removeEventListener('resize', this.$calcTableHeight())
            window.removeEventListener('resize', this.showDetailBtnFun)
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
            .table-btn {
                p.period-link {
                    position: absolute;
                    bottom: 10px;
                    left: 205px;
                    z-index: 999;
                    color: #ff0036;
                }
            }
        }
    }
</style>
