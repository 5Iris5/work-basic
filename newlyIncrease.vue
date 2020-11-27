<template>
    <div class="content">
        <div class="content-title">
            <!-- <h3>凭证</h3> -->
            <el-form :model="searchForm" inline>
                <el-row>
                    <el-form-item label="帐套">
                        <el-select
                            v-model.number="searchForm.accountBooksId"
                            placeholder="请选择帐套"
                            class="filter-item"
                            size="mini"
                            :disabled="isEditAccount"
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
                </el-row>
            </el-form>
        </div>
        <div class="content-table">
            <div class="btn-top">
                <div class="btn-left">
                    <el-button
                        type="primary"
                        w-90
                        size="mini"
                        v-show="isShowAdd"
                        @click="saveVoucher('add', 2)"
                        :disabled="disabledBtn.disabledAdd"
                        >{{ addsave }}</el-button
                    >
                    <el-button
                        type="primary"
                        w-90
                        size="mini"
                        @click="saveVoucher('save', 2)"
                        :disabled="disabledBtn.disabledSava"
                        >保存</el-button
                    >
                    <el-button
                        type="primary"
                        w-90
                        size="mini"
                        @click="saveVoucher('temSave', 1)"
                        :disabled="disabledBtn.disabledTemSava"
                        v-show="isShowTempSava"
                        >暂存</el-button
                    >
                    <!-- :disabled="disabledBtn.disabledTemSava"
                        v-show="isShowTempSava" -->
                    <el-button
                        type="primary"
                        w-90
                        size="mini"
                        @click="handleCasher"
                        :disabled="disabledBtn.disabledCasher"
                        >现金流</el-button
                    >
                    <el-button type="primary" w-90 size="mini" v-show="isShowDelate" @click="delateVou" disabled
                        >删除</el-button
                    >
                    <el-button
                        type="primary"
                        w-90
                        size="mini"
                        v-show="isShowCheck"
                        @click="checkVoucher('check')"
                        :disabled="disabledBtn.disabledCheck"
                        >审核</el-button
                    >
                    <el-button type="primary" w-90 size="mini" v-show="isShowUncheck" @click="checkVoucher('uncheck')"
                        >反审核</el-button
                    >
                    <el-button w-90 size="mini" v-show="isShowBack" @click="backPage">返回</el-button>
                </div>
                <div class="btn-right">
                    <el-button w-90 size="mini" @click="getListForPag('prev')" :disabled="prevDisabled"
                        >上一张</el-button
                    >
                    <el-button w-90 size="mini" @click="getListForPag('next')" :disabled="nextDisabled"
                        >下一张</el-button
                    >
                </div>
            </div>
            <div class="btn-bottom">
                <div class="input-btn">
                    <el-input
                        placeholder="请输入凭证号"
                        v-model.number="voucherNumber"
                        size="mini"
                        clearable
                        :disabled="isEditAccount"
                        @change="changeVoucherNumber"
                    ></el-input>
                    <el-date-picker
                        class="date-picker"
                        type="date"
                        v-model="businessDate"
                        placeholder="选择日期"
                        format="yyyy-MM-dd "
                        value-format="yyyy-MM-dd"
                        size="mini"
                        clearable
                        @change="getVouNo('init')"
                        :disabled="isEditAccount"
                    >
                    </el-date-picker>
                </div>
                <!-- <span>2020年第七期</span> -->
                <p>记账凭证</p>
            </div>
            <editTable
                :thlabel="thlabel"
                :datat="tableData"
                :isEdit="isEdit"
                :spriteStyle="spriteStyle"
                :totalAmountCap="totalAmountCap"
                :totalTrsamtd="totalTrsamtd"
                :totalTrsamtc="totalTrsamtc"
                @handle-focus-input="handleFocusInput"
                v-model="disabledBtn"
                :cashierInfo="cashierInfo"
                :isEditAccount="isEditAccount"
                :accountBooksId="accountBooksId"
                ref="editTable"
            ></editTable>
        </div>
        <el-dialog
            title="现金流项目指定"
            :visible.sync="cashVisible"
            width="40%"
            :show-close="false"
            :close-on-click-modal="false"
            :close-on-press-escape="false"
            center
        >
            <el-form :model="cashForm" class="cash-form" label-width="100px" size="mini">
                <el-form-item label="现金流科目">
                    <el-input v-model="cashForm.subject" disabled></el-input>
                </el-form-item>
                <el-form-item label="币别">
                    <el-input v-model="cashForm.currency" disabled></el-input>
                </el-form-item>
                <el-form-item label="金额">
                    <el-input v-model="cashForm.amount" disabled></el-input>
                </el-form-item>
            </el-form>
            <el-table :data="cashGrid" border style="width: 100%" size="mini">
                <el-table-column property="subject" label="对方科目分录"></el-table-column>
                <el-table-column property="cashname" label="现金流名称"></el-table-column>
                <el-table-column property="amount" label="金额"></el-table-column>
                <el-table-column label="操作" width="100">
                    <template slot-scope="scope">
                        <el-button @click.native.prevent="addCash(scope.$index, cashGrid)" type="text" size="mini">
                            添加
                        </el-button>
                        <el-button @click.native.prevent="delateCash(scope.$index, cashGrid)" type="text" size="mini">
                            移除
                        </el-button>
                    </template>
                </el-table-column>
            </el-table>
            <div slot="footer" class="dialog-footer">
                <el-button @click="cancelCashDialog" size="mini" w-65>取 消</el-button>
                <el-button type="primary" @click="cofirmCashDialog" size="mini" w-65>确 定</el-button>
            </div>
        </el-dialog>
    </div>
</template>
<script>
import editTable from '@/components/voucher/editTable'
import { findAllAccountBook } from '@/api/fims/setSubject'
import { findVouNo, saveVou, updateVoucher, findAllVou, deleteBy, findVoucherPage } from '@/api/fims/voucherSelect'
import { formatDate, sum, parseFormatNum, delCommafy, ChineseNumber, copyArray, deepCopy } from '@/utils/index'

export default {
    inject: ['reload'],
    name: 'newlyIncrease',
    components: {
        editTable,
    },
    data() {
        return {
            thlabel: [
                [
                    { prop: 'brief', label: '摘要' },
                    { prop: 'accountSubject', label: '会计科目' },
                    { prop: 'trsamtd', label: '借方金额' },
                    { prop: 'trsamtc', label: '贷方金额' },
                ],
            ],
            tableData: [],
            // 按钮状态
            disabledBtn: {
                disabledAdd: true,
                disabledSava: true,
                disabledTemSava: true,
                disabledCheck: false,
                disabledCasher: true,
            },
            isShowAdd: true,
            isShowTempSava: true,
            isShowDelate: false,
            isShowCheck: false,
            isShowUncheck: false,
            isShowBack: false,
            // 帐套
            searchForm: {
                accountBooksId: 1,
            },
            accountBookList: [],
            // 凭证资料
            voucherNumber: '',
            businessDate: '',
            status: '',
            addsave: '保存并新增',
            voucherId: '',
            isEdit: false,
            isEditAccount: false,
            cashierInfo: {},
            isVouType: '',
            recSpNo: '',
            voucherType: 1,
            prevDisabled: false,
            nextDisabled: false,
            backParmas: {},
            accountBooksId: 1,
            recSubPayMentId: '',
            cashVisible: false,
            cashForm: {},
            cashGrid: [
                {
                    subject: '对方科目分录一',
                    cashname: '现金流一',
                    amount: '金额一',
                },
                {
                    subject: '对方科目分录二',
                    cashname: '现金流二',
                    amount: '金额二',
                },
                {
                    subject: '对方科目分录三',
                    cashname: '现金流三',
                    amount: '金额三',
                },
            ],
            recBillId: '',
            taskReceiveId: '',
            businessPayableId: '',
            profitId: '',
            spriteStyle: false,
            initDatat: [],
            entryAllIds: [],
            maxPage: 0,
        }
    },
    computed: {
        // 合计
        totalTrsamtd() {
            let borrowSum = []
            borrowSum = this.tableData.map(v => Number(delCommafy(v.trsamtd)))
            let _sum = sum(borrowSum)
            return parseFormatNum(_sum, 2)
        },
        totalTrsamtc() {
            let loanSum = []
            loanSum = this.tableData.map(v => Number(delCommafy(v.trsamtc)))
            let _sum = sum(loanSum)
            return parseFormatNum(_sum, 2)
        },
        totalAmountCap() {
            if (this.totalTrsamtd === '0.00') {
                return ''
            }
            let chineseNum = Number(delCommafy(this.totalTrsamtd))
            return ChineseNumber(chineseNum)
        },
        // 过滤tableData内部的空数据
        splitTableData() {
            let newArray = this.tableData.filter(v => {
                for (let key in v) {
                    if (v[key]) {
                        return v
                    }
                }
            })
            return newArray
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
    watch: {
        accountBooksId() {
            this.accountBooksId = this.searchForm.accountBooksId
        },
    },
    created() {
        if (this.changeIsInitEnter) {
            // console.log('初始页面进入changeIsInitEnter...', this.changeIsInitEnter)
            this.copyData(9)
            this.getDate()
            this.getVouNo('init')
            this.getAllAccountBook()
        } else {
            // console.log('查看凭证进入changeIsInitEnter...', this.changeIsInitEnter)
            this.enterPathByOther()
        }
    },
    beforeRouteLeave(to, from, next) {
        if (from.fullPath === '/fims/voucher/newly') {
            this.$store.state.isInitEnter = true
            this.$store.state.isInitAccBook = true
        }
        // console.log('是否从菜单页面点击进入： ', this.$store.state.isInitEnter)
        next()
    },
    methods: {
        // 现金流
        handleCasher() {
            this.cashVisible = true
            console.log(this.tableData)
        },
        addCash(index, list) {
            let addEmptyObj = {
                subject: '',
                cashname: '',
                amount: '',
            }
            list.push(addEmptyObj)
            console.log(index, list)
        },
        delateCash(index, list) {
            list.splice(index, 1)
            console.log(index, list)
        },
        cancelCashDialog() {
            this.cashVisible = false
        },
        cofirmCashDialog() {
            this.cashVisible = false
        },

        // 翻页功能
        async getListForPag(type) {
            // console.log(type, this.businessDate)
            let option = type === 'prev' ? 1 : type === 'next' ? 2 : ''
            let pagemsg = {
                accountBooksId: this.searchForm.accountBooksId,
                voucherNumber: this.voucherNumber,
                option: option,
            }
            if (this.periodId) {
                pagemsg = { ...pagemsg, periodId: this.periodId }
            } else {
                pagemsg = { ...pagemsg, businessDate: this.businessDate }
            }
            let { data } = await findVoucherPage({
                ...pagemsg,
            })
            if (data.ret === 1) {
                this.isVouType = 'pagination'
                let voucher = data.data
                let maxPage = data.data.totalCount
                if (maxPage === 0) {
                    if (type === 'prev') {
                        this.prevDisabled = true
                        this.$message.warning('已经是第一张凭证了')
                    } else if (type === 'next') {
                        this.nextDisabled = true
                        this.$message.warning('已经是最后一张凭证了')
                    }
                } else {
                    this.nextDisabled = false
                    this.prevDisabled = false
                    this.$nextTick(() => {
                        this.disabledBtn.disabledAdd = true
                        this.disabledBtn.disabledSava = true
                        this.disabledBtn.disabledTemSava = true
                        this.getVouData(voucher)
                    })
                    // this.$message.success(data.msg)
                }
            } else {
                this.$message.error(data.msg)
            }
        },

        // 初始化数据
        getEditAccBook(accountBooksId, businessDate) {
            this.copyData(9)
            this.getDate(businessDate)
            this.getVouNo(accountBooksId)
            this.getAllAccountBook(accountBooksId)
            this.$refs.editTable.getAllSubject(accountBooksId)
        },

        // vuex
        enterPathByOther() {
            let data = this.setVoucherTempData
            if (data) {
                this.isVouType = data.type
                this.isShowBack = data.backStatus
                if (data.backStatus) {
                    this.isShowAdd = data.addStatus
                }
                if (data.type === 'add') {
                    this.$nextTick(() => {
                        // console.log('帐套id： ' + data.accountBooksId)
                        this.searchForm.accountBooksId = data.accountBooksId
                        this.getEditAccBook(data.accountBooksId)
                    })
                } else if (data.type === 'edit') {
                    this.$nextTick(() => {
                        // let voucher = data.checkedVou[0]
                        this.getVouData(data)
                    })
                } else {
                    this.isShowTempSava = data.isShowTempSava
                    if (data.type === 'receipt') {
                        this.recSpNo = data.spNo
                    } else if (data.type === 'payment') {
                        this.recSubPayMentId = data.subPayMentId
                    } else if (data.type === 'bill') {
                        this.recBillId = data.receiptId
                    } else if (data.type === 'taskReceive') {
                        this.taskReceiveId = data.taskReceiveId
                    } else if (data.type === 'profit') {
                        this.profitId = data.profitId
                    } else if (data.type === 'businessPayable') {
                        this.businessPayableId = data.businessPayableId
                    }

                    // this.voucherType = data.voucherType
                    this.isEditAccount = data.isEditAccount
                    this.$nextTick(() => {
                        this.getList()
                    })
                }
            }
        },

        // 查询凭证  编辑 -> voucherId: this.editVoucherId,
        async getList() {
            let { data } =
                this.isVouType === 'receipt'
                    ? await findAllVou({
                          pageNo: 1,
                          pageSize: 10,
                          spNo: this.recSpNo,
                      })
                    : this.isVouType === 'payment'
                    ? await findAllVou({
                          pageNo: 1,
                          pageSize: 10,
                          //   subPayMentId: this.recSubPayMentId,
                          voucherId: this.recSubPayMentId,
                      })
                    : this.isVouType === 'bill'
                    ? await findAllVou({
                          pageNo: 1,
                          pageSize: 10,
                          //   receiptId: this.recBillId,
                          voucherId: this.recBillId,
                      })
                    : this.isVouType === 'taskReceive'
                    ? await findAllVou({
                          pageNo: 1,
                          pageSize: 10,
                          voucherId: this.taskReceiveId,
                      })
                    : this.isVouType === 'profit'
                    ? await findAllVou({
                          pageNo: 1,
                          pageSize: 10,
                          voucherId: this.profitId,
                      })
                    : this.isVouType === 'businessPayable'
                    ? await findAllVou({
                          pageNo: 1,
                          pageSize: 10,
                          voucherId: this.businessPayableId,
                      })
                    : await findAllVou({
                          pageNo: 1,
                          pageSize: 10,
                      })
            if (data.ret === 1) {
                let voucher = data.data
                console.log(voucher)
                this.$nextTick(() => {
                    this.getVouData(voucher)
                })
            }
        },
        // 所查凭证信息
        getVouData(data) {
            let voucher
            if (this.isVouType === 'edit') {
                voucher = data.checkedVou[0]
            } else {
                voucher = data.list[0]
            }
            // console.log('form: ' + data.type, data, voucher)
            // 制单人等信息
            this.cashierInfo = {
                createZh: voucher.createZh,
                checkZh: voucher.checkZh,
                posterZh: voucher.posterZh,
                cashierZh: voucher.cashierZh,
            }
            // console.log(this.cashierInfo)
            let initVoucher = voucher.entryInfo
            let len = voucher.entryInfo.length
            // console.log(initVoucher, len)
            this.accountBooksId = voucher.accountBooksId
            this.businessDate = voucher.businessDate
            this.periodId = voucher.periodId
            // console.log('时间：' + this.businessDate)
            this.voucherNumber = voucher.voucherNumber
            this.voucherId = voucher.voucherId
            // 判断精灵图什么时候出现？
            console.log(data, voucher.status, data.isEdit)
            this.isEdit = voucher.status === 3 || voucher.status === 5 ? true : false
            this.isEditAccount = this.isEdit
            if (voucher.status === 1) {
                // 暂存
                this.disabledBtn.disabledSava = this.isShowCheck = this.isShowUncheck = false
                console.log(this.isShowCheck)
            } else if (voucher.status === 2) {
                // 保存
                this.isShowDelate = data.deleteStatus ? data.deleteStatus : true
                this.isShowCheck = data.checkStatus ? data.checkStatus : true
            } else if (voucher.status === 3) {
                // 审核
                this.isShowUncheck = data.uncheckStatus ? data.uncheckStatus : true
                this.spriteStyle = false
            } else if (voucher.status === 5) {
                // 已结账  ->  页面不可编辑,上下张可
                this.spriteStyle = this.disabledBtn.disabledCasher = true
                this.isShowDelate = this.isShowCheck = this.isShowUncheck = false
            }
            // 凭证类型
            this.voucherType = voucher.voucherType
            // console.log('凭证类型：' + this.voucherType)
            // 重构数据 调用帐套与科目
            this.resetData(initVoucher, len)
            this.getAllAccountBook(voucher.accountBooksId)
            // console.log(this.changeIsInitEnter)
            this.$refs.editTable.getAllSubject(voucher.accountBooksId)
        },
        // 重构数据 金额转义/补齐七行
        resetData(initVoucher, len) {
            // 金额转义
            let tableData = initVoucher.map(v => {
                let key = 'showAccountSubject'
                let value = false
                v[key] = value
                if (v.trsamtd === null || v.trsamtd === 0) {
                    v.trsamtd = ''
                } else {
                    v.trsamtd = parseFormatNum(Number(v.trsamtd), 2)
                }
                if (v.trsamtc === null || v.trsamtc === 0) {
                    v.trsamtc = ''
                } else {
                    v.trsamtc = parseFormatNum(Number(v.trsamtc), 2)
                }
                // console.log(v.accountSubject)
                return v
            })
            // console.log(tableData)
            // 补齐七行
            let emptyObj = []
            if (len < 9) {
                let lackLen = 9 - len
                let obj = [
                    {
                        brief: '',
                        accountSubject: '',
                        trsamtd: '',
                        trsamtc: '',
                        subjectId: '',
                        assistAccountIds: '',
                        showAccountSubject: false,
                    },
                ]
                emptyObj = copyArray(lackLen, obj)
                // console.log(emptyObj)
            }
            this.tableData = tableData.concat(emptyObj)
            // console.log(this.tableData)

            this.initDatat = deepCopy(this.tableData)
            let entryAllIds = this.initDatat.map(item => item.urrId).filter(item => item)
            this.entryAllIds = entryAllIds
            // console.log('原始数据： ', this.initDatat, '分录id： ', this.entryAllIds)
        },

        // 审核/反审核数据
        async checkVoucher(type) {
            let { data } = await updateVoucher({
                optionType: type,
                voucherIds: this.voucherId,
            })
            if (data.ret === 1) {
                if (type === 'check') {
                    this.isShowCheck = false
                    this.isShowUncheck = true
                    this.isEdit = true
                    this.isEditAccount = true
                    this.disabledBtn.disabledCasher = true
                    this.spriteStyle = false
                } else {
                    this.isShowCheck = true
                    this.isShowUncheck = false
                    this.isEdit = false
                    this.isEditAccount = false
                }
                this.$message.success(data.msg)
            } else {
                this.$message.error(data.msg)
            }
        },
        // 删除凭证
        async delateVou() {
            let { data } = await deleteBy({
                voucherIds: [this.voucherId],
            })
            if (data.ret === 1) {
                this.$message.success(data.msg)
                // 调翻页接口，加载新凭证
            } else {
                this.$message.error(data.msg)
            }
        },

        // 复制数据
        copyData(num) {
            // 摘要，会计科目，借方，贷方，科目id，辅助核算id，控制select出现状态
            let tableData = [
                {
                    brief: '',
                    accountSubject: '',
                    trsamtd: '',
                    trsamtc: '',
                    subjectId: '',
                    assistAccountIds: '',
                    showAccountSubject: false,
                },
            ]
            this.tableData = copyArray(num, tableData)
            // console.log(this.tableData.length)
        },
        // 帐套
        async getAllAccountBook(value) {
            let { data } = await findAllAccountBook({
                pageNo: 1,
                pageSize: 200,
            })
            if (data.ret === 1) {
                this.accountBookList = data.data.list
                this.searchForm.accountBooksId = value ? value : this.accountBookList[0].accountBooksId
            } else {
                this.$message.error(data.msg)
            }
        },
        // 修改帐套，凭证号，日期触发按钮状态
        handleKeyDownCell() {
            this.disabledBtn.disabledAdd = false
            this.disabledBtn.disabledSava = false
            this.disabledBtn.disabledTemSava = false
            this.disabledBtn.disabledCheck = true
        },
        changeVoucherNumber() {
            this.handleKeyDownCell()
        },
        // 改变帐套 -> 重新获取凭证账号与会计科目
        changeAccountBook(value) {
            this.searchForm.accountBooksId = value
            this.getVouNo('init')
            this.$refs.editTable.getAllSubject(value)
            this.handleKeyDownCell()
        },
        // 默认显示当前日期
        getDate(value) {
            let businessDate = new Date()
            this.businessDate = value ? value : formatDate(businessDate, 'yyyy-MM-dd')
        },
        // 获取会计期间凭证号
        async getVouNo(value) {
            // console.log('会计期间凭证号：', value, '修改后的日期值：', this.businessDate)
            let { data } =
                value === 'init'
                    ? await findVouNo({
                          dataTime: this.businessDate,
                          accountBooksId: this.searchForm.accountBooksId,
                      })
                    : await findVouNo({
                          dataTime: this.businessDate,
                          accountBooksId: value ? value : this.searchForm.accountBooksId,
                      })
            data.ret === 1 ? (this.voucherNumber = data.data) : this.$message.error(data.msg)
            this.handleKeyDownCell()
        },

        // 保存新增凭证
        async postVouData(type, status) {
            let splitTableData = this.splitTableData.map(v => {
                let newV = deepCopy(v)
                if (newV.trsamtd) {
                    newV.trsamtd = Number(delCommafy(newV.trsamtd))
                }
                if (newV.trsamtc) {
                    newV.trsamtc = Number(delCommafy(newV.trsamtc))
                }
                return newV
            })

            // console.log(this.tableData, splitTableData)
            let entrySplitIds = splitTableData.map(item => item.urrId).filter(item => item)
            // console.log('总凭证id：', this.entryAllIds, '现存凭证id： ', entrySplitIds)
            let entryDeleteIds = []
            this.entryAllIds.forEach((item, index) => {
                // console.log('item: ', item, 'index: ', index, entrySplitIds[index])
                if (entrySplitIds.indexOf(this.entryAllIds[index]) === -1) {
                    // console.log(index, this.entryAllIds[index])
                    entryDeleteIds.push(this.entryAllIds[index])
                }
            })
            // console.log('删除凭证分录id: ', entryDeleteIds)

            let savaData = {
                voucherNumber: this.voucherNumber,
                businessDate: this.businessDate,
                accountBooksId: this.searchForm.accountBooksId,
                status: status,
                totalAmountCap: this.totalAmountCap,
                totalTrsamtd: Number(delCommafy(this.totalTrsamtd)),
                totalTrsamtc: Number(delCommafy(this.totalTrsamtc)),
                entryInfo: [...splitTableData],
                voucherType: this.voucherType,
            }
            if (entryDeleteIds.length > 0) {
                savaData = { ...savaData, entryDeleteIds: entryDeleteIds }
            }
            let { data } = this.voucherId
                ? await saveVou({
                      ...savaData,
                      voucherId: this.voucherId,
                  })
                : await saveVou({
                      ...savaData,
                  })
            if (data.ret === 1) {
                this.$message.success(data.msg)
                if (type === 'add') {
                    // this.reload()
                    // console.log('新增')
                    this.getEditAccBook(this.searchForm.accountBooksId, this.businessDate)
                }
                if (type === 'save') {
                    this.disabledBtn.disabledCheck = false
                }
            } else {
                this.$message.error(data.msg)
            }
        },
        deleteVoucherPartByCell(DataList) {
            return new Promise(resolve => {
                DataList = DataList.map(v => {
                    if (v.brief === '' && v.accountSubject === '' && v.trsamtd === '' && v.trsamtc === '') {
                        for (let key in v) {
                            if (v[key]) {
                                v[key] = ''
                            }
                        }
                    }
                    return v
                })
                // console.log('通过表格手动删除后的凭证分录：', DataList)
                resolve(DataList)
            })
        },
        checkedVoucherBeforeSava(type, status) {
            // console.log('保存', this.tableData, type, status)
            let isReturn = false
            this.tableData.forEach((v, idx) => {
                for (let key in v) {
                    if (v[key]) {
                        if (!v.accountSubject) {
                            if (!isReturn) {
                                this.$message({
                                    message: `第${idx + 1}行，请选择科目`,
                                    type: 'warning',
                                    duration: 3500,
                                })
                            }
                            isReturn = true
                        }
                    }
                }
            })
            if (isReturn) return
            // 新增按钮
            if (type === 'add' && this.addsave === '新增') {
                if (this.disabledBtn.disabledSava === true && this.disabledBtn.disabledTemSava === true) {
                    // this.reload()
                    // console.log('新增')
                    this.getEditAccBook(this.searchForm.accountBooksId, this.businessDate)
                    return
                } else {
                    this.$confirm('检测到您已修改了凭证，是否在新增前保存修改？', '确认信息', {
                        showClose: false,
                        distinguishCancelAndClose: true,
                        confirmButtonText: '保存',
                        cancelButtonText: '放弃修改',
                    })
                        .then(() => {
                            // console.log('编辑后新增，走暂存逻辑...')
                            // this.postVouData('add', 1)
                            // console.log('编辑后新增，走保存逻辑...')
                            this.checkEditSave(isReturn).then(() => {
                                // console.log('isReturn', isReturn)
                                if (isReturn) return
                                this.postVouData('add', 2)
                            })
                        })
                        .catch(() => {
                            // this.reload()
                            // console.log('新增前保存修改')
                            this.getEditAccBook(this.searchForm.accountBooksId, this.businessDate)
                            return
                        })
                }
            }
            if (type === 'temSave') {
                this.addsave = '新增'
                this.disabledBtn.disabledSava = true
                this.disabledBtn.disabledTemSava = true
                this.postVouData(type, status)
            } else if (type === 'save' || this.addsave === '保存并新增') {
                // console.log(type, 'sava保存,add保存并新增')
                let isReturn = false
                this.tableData.forEach((v, idx) => {
                    for (let key in v) {
                        if (v[key]) {
                            if (!v.brief) {
                                if (!isReturn) {
                                    this.$message({
                                        message: `第${idx + 1}行，请输入摘要`,
                                        type: 'warning',
                                        duration: 3500,
                                    })
                                }
                                isReturn = true
                            }
                            if (!v.trsamtd && !v.trsamtc) {
                                if (!isReturn) {
                                    this.$message({
                                        message: `第${idx + 1}行，请输入金额`,
                                        type: 'warning',
                                        duration: 3500,
                                    })
                                }
                                isReturn = true
                            }
                        }
                    }
                })
                if (isReturn) return
                if (this.totalTrsamtd !== this.totalTrsamtc) {
                    this.$message({
                        message: '请先确保借贷平衡，再进行保存！',
                        type: 'warning',
                        duration: 3500,
                    })
                    return
                }
                if (type === 'save') {
                    // console.log('sava保存')
                    this.addsave = '新增'
                    this.disabledBtn.disabledSava = true
                    this.disabledBtn.disabledTemSava = true
                }
                this.postVouData(type, status)
            }
        },
        saveVoucher(type, status) {
            this.deleteVoucherPartByCell(this.tableData).then(() => {
                this.checkedVoucherBeforeSava(type, status)
            })
        },

        // 修改后保存的校验
        checkEditSave() {
            return new Promise(resolve => {
                let isReturn = false
                this.tableData.forEach((v, idx) => {
                    for (let key in v) {
                        if (v[key]) {
                            if (!v.brief) {
                                if (!isReturn) {
                                    this.$message({
                                        message: `第${idx + 1}行，请输入摘要`,
                                        type: 'warning',
                                        duration: 3500,
                                    })
                                }
                                isReturn = true
                            }
                            if (!v.trsamtd && !v.trsamtc) {
                                if (!isReturn) {
                                    this.$message({
                                        message: `第${idx + 1}行，请输入金额`,
                                        type: 'warning',
                                        duration: 3500,
                                    })
                                }
                                isReturn = true
                            }
                        }
                    }
                })
                if (isReturn) return
                if (this.totalTrsamtd !== this.totalTrsamtc) {
                    this.$message({
                        message: '请先确保借贷平衡，再进行保存！',
                        type: 'warning',
                        duration: 3500,
                    })
                    return
                }
                resolve(isReturn)
            })
        },

        /*backPage() {
            this.$store.commit('changeIsNewlyIncreaseEnter', true)
            this.$store.commit('changeIsInitEnter', true)
            if (this.isVouType === 'receipt') {
                this.$router.push('/fims/receipt/recExpense')
            } else if (this.isVouType === 'payment') {
                this.$router.push('/fims/receipt/recPayment')
            } else if (this.isVouType === 'bill') {
                this.$router.push('/fims/receipt/recBill')
            } else if (this.isVouType === 'taskReceive') {
                this.$router.push('/fims/receiveManage/taskReceive')
            } else if (this.isVouType === 'add' || this.isVouType === 'edit') {
                this.$router.push('/fims/voucher/examine')
            }
        },*/
        backPage() {
            this.$store.commit('changeIsNewlyIncreaseEnter', true)
            this.$store.commit('changeIsInitEnter', true)
            this.$router.go(-1)
        },

        // 控制select出现
        handleFocusInput() {
            this.tableData = this.tableData.map(item => {
                return { ...item, showAccountSubject: false }
            })
        },
    },
}
</script>
<style lang="less" scoped>
.content {
    overflow-y: scroll;
    height: 100%;
    overflow-x: hidden;
    .content-title {
        background: #ffffff;
        padding: 5px 30px 0px 27px;
        box-sizing: border-box;
        box-shadow: 0px 0px 5px 0px #d9d9d9;
        // height: 99px;
        overflow: hidden;
        h3 {
            color: #666666;
            font-size: 16px;
            font-weight: 400;
        }
        /deep/.el-select {
            input {
                text-align: left;
            }
        }
        .el-form-item {
            margin-right: 0;
            margin-bottom: 5px;
        }
    }
    .content-table {
        margin: 10px;

        padding-bottom: 30px;
        // height: calc(100% - 100px);
        box-shadow: -2px 0px 10px 0px #d9d9d9;
        background-color: #fff;
        .btn-top {
            padding: 30px 40px 0px;
            .btn-left {
                float: left;
            }
            .btn-right {
                float: right;
            }
            &::after {
                content: '';
                display: block;
                clear: both;
            }
        }
        .btn-bottom {
            padding: 20px 40px 0;
            .input-btn {
                float: left;
                margin-bottom: 10px;
                /deep/.el-input {
                    display: inline-block;
                    width: 120px;
                    margin-right: 10px;
                }
            }
            p {
                float: left;
                text-align: center;
                font-size: 16px;
                width: calc((100%-340px) / 2);
                span {
                    margin-left: 30px;
                    font-size: 14px;
                }
            }
            &::after {
                content: '';
                display: block;
                clear: both;
            }
            /deep/.date-picker {
                width: 160px !important;
            }
        }
    }
    /deep/.cash-form {
        width: 50%;
    }
}
</style>
