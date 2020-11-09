<template>
    <div class="content-box">
        <div class="table-box">
            <el-button type="primary" @click="editOne('add')" w-90 size="mini">新增</el-button>
            <el-button type="primary" @click="exportFile" w-90 size="mini" disabled>导出</el-button>
            <el-button @click="deleteOne('some')" :disabled="selList.length === 0" size="mini" w-90>批量删除</el-button>
            <el-button size="mini" @click="reload()" class="fr" w-110>重置列宽</el-button>
            <div class="table-content">
                <el-table
                    :data="tableData.list"
                    @selection-change="selectChange"
                    border
                    stripe
                    header-cell-class-name="tableHeader"
                    size="mini"
                    height="calc(100% - 94px)"
                    row-key="id"
                    :tree-props="{ children: 'children' }"
                    :default-expand-all="true"
                    @select="handleSelect"
                    @select-all="handleSelectAll"
                    ref="multipleTable"
                >
                    <el-table-column type="selection"></el-table-column>
                    <el-table-column prop="subjectCode" label="科目代码" width="180px"> </el-table-column>
                    <el-table-column prop="subjectName" label="科目名称" width="200px"> </el-table-column>
                    <el-table-column prop="subjectType" label="科目类别" width="120"> </el-table-column>
                    <el-table-column prop="balanceDirect" label="余额方向" width="80"> </el-table-column>
                    <el-table-column prop="assistAccountZh" label="辅助核算" width="360"> </el-table-column>
                    <el-table-column prop="foreiCurrAccZh" label="外币核算" width="90"> </el-table-column>
                    <el-table-column prop="endRemittance" label="期末调汇" width="80"> </el-table-column>
                    <!-- <el-table-column prop="fullName" label="全名"> </el-table-column> -->
                    <el-table-column prop="cashSubject" label="现金科目" width="80"> </el-table-column>
                    <el-table-column prop="bankSubject" label="银行科目" width="80"> </el-table-column>
                    <el-table-column prop="contactsBusiness" label="往来业务核算" width="100"> </el-table-column>
                    <el-table-column prop="status" label="状态" width="80"> </el-table-column>
                    <el-table-column label="操作" width="120">
                        <template slot-scope="scope">
                            <el-button
                                small="small"
                                type="text"
                                @click="editOne('one', scope.row)"
                                v-show="scope.row.isShowAdd"
                                style="margin-left: 0"
                                >新增</el-button
                            >
                            <el-button
                                small="small"
                                type="text"
                                @click="editOne('edit', scope.row)"
                                style="margin-left: 0"
                                >编辑</el-button
                            >
                            <el-button
                                small="small"
                                type="text"
                                @click="deleteOne('one', scope.row)"
                                v-show="scope.row.isChildren"
                                style="margin-left: 0"
                                >删除</el-button
                            >
                        </template>
                    </el-table-column>
                </el-table>
                <Pagination
                    :pageNumber="pageNumber"
                    :pageSize="pageSize"
                    :totalCount="tableData.totalCount"
                    :totalPage="tableData.totalPage"
                    @go-page="goPage"
                    @change-size="changeSize"
                ></Pagination>
            </div>
        </div>
        <el-dialog
            :title="isEdit ? '编辑科目信息' : '新增科目信息'"
            :visible.sync="dialogVisible"
            width="40%"
            @close="cancelMethod"
        >
            <el-form :model="form" label-width="110px" :rules="formRule" ref="form" :show-message="false">
                <div class="form-fl-1">
                    <div class="form-l">
                        <!-- <el-form-item label="科目代码" prop="subjectCode">
                            <el-input
                                v-model.trim="form.subjectCode"
                                placeholder="请输入科目编码"
                                size="mini"
                            ></el-input>
                        </el-form-item> -->
                        <el-form-item
                            class="subject-code"
                            label="科目代码"
                            prop="subjectCode"
                            v-if="clickType === 'add'"
                        >
                            <div class="add">
                                <el-input
                                    class="subject-code-edit"
                                    v-model.trim="form.subjectCode"
                                    placeholder=""
                                    size="mini"
                                    maxlength="4"
                                ></el-input>
                            </div>
                        </el-form-item>
                        <el-form-item
                            :class="['subject-code', { diseditActive: isEditActive }]"
                            label="科目代码"
                            prop="codeEdit"
                            v-else
                        >
                            <div class="one" v-if="clickType === 'one'">
                                <span class="subject-code-fixed">{{ codeOnly }}</span>
                                <el-input
                                    class="subject-code-edit"
                                    v-model.trim="form.codeEdit"
                                    placeholder=""
                                    size="mini"
                                    maxlength="2"
                                ></el-input>
                            </div>
                            <div class="edit" v-else>
                                <span class="subject-code-fixed">{{ codeOnlyPrev }}</span>
                                <el-input
                                    class="subject-code-edit"
                                    v-model.trim="form.codeEdit"
                                    placeholder=""
                                    size="mini"
                                    :maxlength="codeMaxLength"
                                    :disabled="editCodeDisabled"
                                ></el-input>
                            </div>
                        </el-form-item>
                        <el-form-item label="上级科目">
                            <el-input
                                v-model.trim="form.parentName"
                                placeholder="请选择上级科目"
                                clearable
                                size="mini"
                                disabled
                            ></el-input>
                        </el-form-item>
                        <el-form-item label="全名">
                            <el-input
                                v-model.trim="fullName"
                                placeholder="请输入全名"
                                clearable
                                size="mini"
                                disabled
                            ></el-input>
                        </el-form-item>
                    </div>
                    <div class="form-r">
                        <el-form-item label="科目名称" prop="subjectName">
                            <el-input
                                v-model.trim="form.subjectName"
                                placeholder="请输入科目名称"
                                clearable
                                size="mini"
                                maxlength="20"
                                show-word-limit
                            ></el-input>
                        </el-form-item>
                        <el-form-item label="科目类别" prop="subjectTypeId">
                            <el-select
                                v-model="form.subjectTypeId"
                                placeholder="请选择科目类别"
                                clearable
                                size="mini"
                                :disabled="subjectTypeDisabled"
                            >
                                <el-option
                                    v-for="item in subjectTypeList"
                                    :key="item.busId"
                                    :label="item.busName"
                                    :value="item.busId"
                                >
                                </el-option>
                            </el-select>
                        </el-form-item>
                    </div>
                </div>
                <div class="form-fl-2">
                    <div class="form-l">
                        <el-form-item label="余额方向" prop="balanceDirect">
                            <el-radio-group v-model="form.balanceDirect" :disabled="balanceDirectDisabled">
                                <el-radio :label="'借'">借</el-radio>
                                <el-radio :label="'贷'">贷</el-radio>
                            </el-radio-group>
                        </el-form-item>
                        <el-form-item label="是否现金科目">
                            <el-radio-group v-model="form.cashSubject" :disabled="cashSubjectDisabled">
                                <el-radio :label="'是'">是</el-radio>
                                <el-radio :label="'否'">否</el-radio>
                            </el-radio-group>
                        </el-form-item>
                    </div>
                    <div class="form-r">
                        <el-form-item label="科目状态" prop="status">
                            <el-radio-group v-model="form.status">
                                <el-radio :label="'启用'">启用</el-radio>
                                <el-radio :label="'禁用'">禁用</el-radio>
                            </el-radio-group>
                        </el-form-item>
                        <el-form-item label="是否银行科目">
                            <el-radio-group v-model="form.bankSubject" :disabled="statusDisabled">
                                <el-radio :label="'是'">是</el-radio>
                                <el-radio :label="'否'">否</el-radio>
                            </el-radio-group>
                        </el-form-item>
                    </div>
                </div>

                <div class="form-bottom">
                    <el-form-item label="辅助核算">
                        <el-checkbox-group v-model="assistAccount" @change="handleCheckedAssistAccount">
                            <el-checkbox v-for="item in accountInfoList" :label="item.key" :key="item.id">{{
                                item.value
                            }}</el-checkbox>
                        </el-checkbox-group>
                    </el-form-item>
                    <el-form-item label="外币核算">
                        <el-radio-group v-model="form.foreiCurrAcc" @change="handleCheckedCurrency">
                            <el-radio v-for="item in currencyInfoList" :key="item.id" :label="item.key * 1">{{
                                item.value
                            }}</el-radio>
                        </el-radio-group>
                    </el-form-item>
                    <el-form-item label="期末调汇" v-show="isShowRemittance">
                        <el-radio-group v-model="form.endRemittance" :disabled="remittanceDisabled">
                            <el-radio :label="'是'">是</el-radio>
                            <el-radio :label="'否'">否</el-radio>
                        </el-radio-group>
                    </el-form-item>
                    <el-form-item label="往来业务核算">
                        <el-radio-group v-model="form.contactsBusiness">
                            <el-radio :label="'是'">是</el-radio>
                            <el-radio :label="'否'">否</el-radio>
                        </el-radio-group>
                    </el-form-item>
                </div>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click="cancelMethod" size="mini" w-65>取 消</el-button>
                <el-button type="primary" @click="saveMethod" size="mini" w-65>确 定</el-button>
            </div>
        </el-dialog>
    </div>
</template>
<script>
import { findAll, add, deleteBy, update, findAllTitle, findCurrency, findSubjectType } from '@/api/fims/setSubject'
import { dealTotastMsg, changeKey } from '@/utils/index'
import Pagination from '@/components/pagination'
import Excel from '@/utils/excel.js'

export default {
    name: 'subjectTab',
    props: {
        activeKey: Number,
        defaultAccountBook: Number,
    },
    data() {
        return {
            dialogVisible: false,
            form: {
                subjectCode: '',
                subjectName: '',
                subjectTypeId: '',
                balanceDirect: '',
                status: '',
                cashSubject: '',
                bankSubject: '',
                foreiCurrAcc: '',
                endRemittance: '',
                contactsBusiness: '',
                parentName: '',
                fullName: '',
                codeEdit: '',
            },
            assistAccount: [],
            searchForm: {},
            tableData: {
                list: [],
            },
            selList: [],
            pageSize: 100,
            pageNumber: 1,
            isEdit: false,
            formRule: {
                codeEdit: [
                    {
                        required: true,
                        trigger: 'blur',
                    },
                ],
                subjectCode: [
                    {
                        required: true,
                        trigger: 'blur',
                    },
                ],
                subjectName: [
                    {
                        required: true,
                        trigger: 'blur',
                    },
                ],
                subjectTypeId: [
                    {
                        required: true,
                        trigger: 'blur',
                    },
                ],
                balanceDirect: [
                    {
                        required: true,
                        trigger: 'blur',
                    },
                ],
                status: [
                    {
                        required: true,
                        trigger: 'blur',
                    },
                ],
            },
            accountInfoList: [],
            currencyInfoList: [],
            subjectTypeList: [],
            subjectTypeDisabled: true,
            remittanceDisabled: true,
            isShowRemittance: true,
            parentId: -1,
            runJoggle: 0,
            excelList: [],
            balanceDirectDisabled: true,
            cashSubjectDisabled: true,
            statusDisabled: true,
            editCodeDisabled: true,
            clickType: '',
            codeLen: '',
            isEditActive: false,
        }
    },
    components: { Pagination },
    computed: {
        fullName() {
            if (
                this.form.fullName === null ||
                this.form.fullName === undefined ||
                this.form.fullName === '' ||
                this.form.fullName.indexOf('-') === -1
            ) {
                if (this.clickType === 'one') {
                    return `${this.form.fullName}-${this.form.subjectName}`
                } else {
                    return this.form.subjectName ? `${this.form.subjectName}` : ''
                }
            } else {
                if (this.clickType === 'one') {
                    return `${this.form.fullName}-${this.form.subjectName}`
                } else {
                    let splitPre = this.form.fullName.substring(0, this.form.fullName.lastIndexOf('-'))
                    return `${splitPre}-${this.form.subjectName}`
                }
            }
        },
        codeOnly() {
            // console.log('this.form.subjectCode: ' + this.form.subjectCode, 'codeOnly: ' + this.codeOnly)
            if (!this.form.subjectCode) return
            if (this.clickType === 'add') return
            if (this.clickType === 'edit') return
            // one
            return this.form.subjectCode
        },
        codeOnlyPrev() {
            if (!this.form.subjectCode) return
            if (this.clickType === 'add') return
            if (this.clickType === 'one') return
            // edit
            if (this.form.subjectCode.indexOf('.') !== -1) {
                let hasChildren = this.form.subjectCode.substring(0, this.form.subjectCode.lastIndexOf('.'))
                return hasChildren + '.'
            } else {
                return ''
            }
        },
        codeMaxLength() {
            if (!this.form.subjectCode) return
            if (this.form.subjectCode.indexOf('.') !== -1) {
                return 2
            } else {
                return 4
            }
        },
        subjectCodeEdit() {
            let newCode = ''
            if (this.codeOnlyPrev) {
                newCode = this.codeOnlyPrev + this.form.codeEdit
            } else {
                newCode = this.form.codeEdit
            }
            return newCode
        },
        subjectCodeOne() {
            let newCode = this.codeOnly + this.form.codeEdit
            return newCode
        },
        subjectCodeAdd() {
            let newCode = this.form.subjectCode
            return newCode
        },
    },
    created() {
        this.getAllTitle()
        this.getAllCurrency()
        this.getSubjectType()
        this.getList()
    },
    methods: {
        async getList(searchForm) {
            // console.log(searchForm)
            this.searchForm = searchForm
            // 控制页码
            if (searchForm) {
                this.pageNumber = 1
            }
            // searchForm = false
            if (this.activeKey === 2 || this.activeKey === 3) {
                this.isShowRemittance = false
            } else {
                this.isShowRemittance = true
            }
            let { data } = await findAll({
                pageNo: this.pageNumber,
                pageSize: this.pageSize,
                subjectBigCategoryId: this.activeKey,
                accountBooksId: this.defaultAccountBook,
                parentId: -1,
                ...searchForm,
            })
            if (data.ret === 1) {
                this.tableData = data.data
                if (this.tableData.list) {
                    this.traverse(this.tableData.list)
                }
                this.runJoggle = Math.ceil(this.tableData.totalCount / 100)
                // console.log(this.runJoggle)
            } else {
                this.$message.error(data.message)
            }
        },
        traverse(arrList) {
            arrList.map(v => {
                v.balanceDirect = v.balanceDirect === 1 ? '借' : '贷'
                v.cashSubject = v.cashSubject === 1 ? '是' : '否'
                v.bankSubject = v.bankSubject === 1 ? '是' : '否'
                v.status = v.status === 1 ? '禁用' : '启用'
                v.contactsBusiness = v.contactsBusiness === 1 ? '是' : '否'
                v.endRemittance = v.endRemittance === 1 ? '是' : '否'
                v.foreiCurrAcc = Number(v.foreiCurrAcc)
                // console.log(v.foreiCurrAcc)
                // console.log(
                //     '余额方向：' + v.balanceDirect,
                //     '现金科目：' + v.cashSubject,
                //     '银行科目: ' + v.bankSubject,
                //     '状态： ' + v.status,
                //     '期末调汇' + v.endRemittance
                //     '往来业务核算：' + v.contactsBusiness,
                // )
                let keyAdd = 'isShowAdd'
                let valueAdd = true
                if (v.subjectCode.length === 13) {
                    valueAdd = false
                }
                v[keyAdd] = valueAdd

                let key = 'isChildren'
                let value
                if (v.children) {
                    value = false
                    this.traverse(v.children)
                } else {
                    value = true
                }
                v[key] = value
            })
        },
        // 数据导出
        async exportFile(isSearchEnter) {
            let allListData = []
            if (!this.runJoggle) {
                allListData = []
                this.exportExcelBy(allListData)
            }
            for (let i = 1; i <= this.runJoggle; i++) {
                let { data } = await findAll({
                    pageNo: i,
                    pageSize: 100,
                    subjectBigCategoryId: this.activeKey,
                    accountBooksId: this.defaultAccountBook,
                    parentId: -1,
                    ...this.searchForm,
                })
                if (data.ret === 1) {
                    if (isSearchEnter) {
                        allListData = allListData.concat(data.data.list)
                        this.exportExcelBy(allListData)
                    } else {
                        allListData = allListData.concat(data.data.list)
                        this.exportExcelBy(allListData)
                    }
                } else {
                    this.$message.error(data.msg)
                }
            }
        },
        exportExcelBy(array) {
            this.excelList = array.map(el => {
                let {
                    // subjectCode,
                    subjectName,
                    subjectType,
                    balanceDirect,
                    assistAccountZh,
                    foreiCurrAccZh,
                    endRemittance,
                    cashSubject,
                    bankSubject,
                    contactsBusiness,
                } = el
                return {
                    // subjectCode,
                    subjectName,
                    subjectType,
                    balanceDirect,
                    assistAccountZh,
                    foreiCurrAccZh,
                    endRemittance,
                    cashSubject,
                    bankSubject,
                    contactsBusiness,
                }
            })
            let keyMap = {
                // subjectCode: '科目代码',
                subjectName: '科目名称',
                subjectType: '科目类别',
                balanceDirect: '余额方向',
                assistAccountZh: '辅助核算',
                foreiCurrAccZh: '外币核算',
                endRemittance: '期末调汇',
                cashSubject: '现金科目',
                bankSubject: '银行科目',
                contactsBusiness: '外来业务核算',
            }
            changeKey(keyMap, this.excelList)

            this.$confirm(`即将导出 ${this.excelList.length} 条数据, 是否继续?`, '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning',
            })
                .then(() => {
                    Excel.exportExcel(this.excelList, '科目')
                    this.$message({
                        type: 'success',
                        message: '导出成功!',
                    })
                })
                .catch(() => {
                    this.$message({
                        type: 'info',
                        message: '已取消导出数据',
                    })
                })
        },

        async getAllTitle() {
            let { data } = await findAllTitle({
                pageNum: 1,
                pageSize: 30,
                busType: 'assistType',
            })
            if (data.ret === 1) {
                this.accountInfoList = data.data.list
            } else {
                this.$message.error(data.msg)
            }
        },
        handleCheckedAssistAccount() {
            this.assistAccountStr = this.assistAccount.join(',')
            // console.log(val, this.assistAccountStr)
        },
        // 外币核算
        async getAllCurrency() {
            let { data } = await findCurrency({
                pageNum: 1,
                pageSize: 20,
                busType: 'foreignAssistType',
            })
            if (data.ret === 1) {
                this.currencyInfoList = data.data.list
            } else {
                this.$message.error(data.msg)
            }
        },
        // 选择外币核算 控制期末调汇
        handleCheckedCurrency(val) {
            // console.log('外币： ' + typeof val, val)
            this.remittanceDisabled = val === 1 ? true : false
        },
        // 科目类别
        async getSubjectType() {
            let { data } = await findSubjectType({
                pageNum: 1,
                pageSize: 30,
                busType: 'subjectType',
            })
            if (data.ret === 1) {
                this.subjectTypeList = data.data.list
            } else {
                this.$message.error(data.msg)
            }
        },

        // 新增/编辑
        editOne(type, row) {
            this.clickType = type
            this.balanceDirectDisabled = type === 'one' ? true : false
            this.cashSubjectDisabled = type === 'one' ? true : false
            this.statusDisabled = type === 'one' ? true : false
            this.editCodeDisabled = type === 'edit' ? true : false
            this.isEditActive = type === 'edit' ? true : false
            this.subjectTypeDisabled = type === 'add' ? false : true
            this.isEdit = type === 'edit' ? true : false
            this.dialogVisible = true
            // console.log(this.clickType, row)
            if (type !== 'edit') {
                this.assistAccount = []
            }
            if (type === 'edit') {
                this.form = { ...row }
                this.parentId = row.id
                if (row.assistAccount) {
                    this.assistAccount = row.assistAccount.split(',')
                }
                // console.log(row.assistAccount, this.assistAccount)
                // 科目代码可编辑部分
                let newCode = this.form.subjectCode
                let codeLen = this.form.subjectCode.length
                // newCode = newCode.substr(codeLen - 2)
                if (this.form.subjectCode.indexOf('.') !== -1) {
                    this.$set(this.form, 'codeEdit', newCode.substr(codeLen - 2))
                } else {
                    this.$set(this.form, 'codeEdit', newCode.substr(codeLen - 4))
                }
                // console.log(
                //     'this.form.subjectCode: ' + this.form.subjectCode,
                //     'newCode: ' + newCode,
                //     'this.form.codeEdit: ' + this.form.codeEdit,
                // )
            } else {
                // 有默认值的选项  Object.assign(this.form,defaultAdd,defaultOne)
                let defaultAdd = {
                    cashSubject: '否',
                    bankSubject: '否',
                    foreiCurrAcc: 1,
                    contactsBusiness: '否',
                }
                let defaultOne = {}
                if (row) {
                    // status: row.status,
                    defaultOne = {
                        bankSubject: row.bankSubject,
                        cashSubject: row.cashSubject,
                        balanceDirect: row.balanceDirect,
                        status: '启用',
                        subjectName: row.subjectName,
                        parentName: row.subjectName,
                        fullName: row.fullName,
                        subjectTypeId: row.subjectTypeId,
                    }
                }
                let form =
                    this.activeKey === 2 || this.activeKey === 3
                        ? {
                              ...defaultAdd,
                              ...defaultOne,
                          }
                        : {
                              ...defaultAdd,
                              ...defaultOne,
                              endRemittance: '否',
                          }
                let addForm =
                    this.activeKey === 2 || this.activeKey === 3
                        ? {
                              ...defaultAdd,
                          }
                        : {
                              ...defaultAdd,
                              endRemittance: '否',
                          }
                if (type === 'one') {
                    this.form.subjectTypeId = row.subjectTypeId
                    if (row.subjectCode.length < 13) {
                        // subjectCode: row.subjectCode + '.01',
                        this.form = {
                            ...form,
                            subjectCode: row.subjectCode + '.',
                        }
                    }
                    this.parentId = row.id
                    // 科目代码可编辑部分
                    if (row.children) {
                        let codeLen = row.children.length + 1
                        if (codeLen < 9) {
                            codeLen = '0' + codeLen
                            this.$set(this.form, 'codeEdit', codeLen)
                        } else {
                            this.$set(this.form, 'codeEdit', codeLen)
                        }
                    } else {
                        this.$set(this.form, 'codeEdit', '01')
                    }
                    // console.log(
                    //     'this.form.subjectCode: ' + this.form.subjectCode,
                    //     'this.form.codeEdit: ' + this.form.codeEdit,
                    // )
                } else {
                    // add
                    this.form = {
                        ...addForm,
                    }
                    this.parentId = -1
                }
            }
            this.form.parentName = type === 'add' ? '没有上级科目' : this.form.parentName
            // 外币核算 -> 期末调汇
            this.remittanceDisabled = this.form.foreiCurrAcc === 1 ? true : false
            // console.log(this.form.foreiCurrAcc, this.remittanceDisabled)
            // this.remittanceDisabled = this.form.endRemittance === '否' ? true : false
            // console.log(row, this.form, this.form.fullName, this.form.parentName)
        },
        changeDataType(row) {
            row.balanceDirect = row.balanceDirect === '借' ? 1 : 2
            row.cashSubject = row.cashSubject === '是' ? 1 : 2
            row.bankSubject = row.bankSubject === '是' ? 1 : 2
            row.status = row.status === '启用' ? 0 : 1
            row.contactsBusiness = row.contactsBusiness === '是' ? 1 : 2
            row.endRemittance = row.endRemittance === '是' ? 1 : 2
            if (row.children) {
                console.log('有子级...')
                this.changeChildrenDateType(row.children)
            }
        },
        changeChildrenDateType(arrList) {
            arrList.map(row => {
                row.balanceDirect = row.balanceDirect === '借' ? 1 : 2
                row.cashSubject = row.cashSubject === '是' ? 1 : 2
                row.bankSubject = row.bankSubject === '是' ? 1 : 2
                row.status = row.status === '启用' ? 0 : 1
                row.contactsBusiness = row.contactsBusiness === '是' ? 1 : 2
                row.endRemittance = row.endRemittance === '是' ? 1 : 2
                if (row.children) {
                    this.changeChildrenDateType(row.children)
                }
            })
        },
        cancelMethod() {
            this.dialogVisible = false
            this.form = {}
            this.assistAccount = []
            this.$refs['form'].resetFields()
        },
        saveMethod() {
            let defaultVid = [
                {
                    key: 'subjectName',
                    msg: '请填写科目名称',
                },
                {
                    key: 'subjectTypeId',
                    msg: '请选择科目类别',
                },
                {
                    key: 'balanceDirect',
                    msg: '请选择余额方向',
                },
                {
                    key: 'status',
                    msg: '请选择科目状态',
                },
            ]
            let validateArr = []
            if (this.clickType === 'add') {
                validateArr = [
                    {
                        key: 'subjectCode',
                        msg: '请填写科目代码',
                    },
                    ...defaultVid,
                ]
            } else {
                validateArr = [
                    {
                        key: 'codeEdit',
                        msg: '请填写科目代码',
                    },
                    ...defaultVid,
                ]
            }
            let result = dealTotastMsg(this.form, validateArr, this)
            if (result) return
            this.$refs['form'].validate(valid => {
                if (valid) {
                    this.changeDataType(this.form)
                    if (this.isEdit) {
                        this.updateProject()
                    } else {
                        this.addProject()
                    }
                    this.cancelMethod()
                }
            })
        },
        async addProject() {
            let { data } =
                this.parentId !== -1
                    ? await add({
                          ...this.form,
                          subjectBigCategoryId: this.activeKey,
                          assistAccount: this.assistAccountStr,
                          fullName: this.fullName,
                          parentId: this.parentId,
                          accountBooksId: this.defaultAccountBook,
                          subjectCode: this.subjectCodeOne,
                      })
                    : await add({
                          // 第一级
                          ...this.form,
                          subjectBigCategoryId: this.activeKey,
                          assistAccount: this.assistAccountStr,
                          fullName: this.fullName,
                          parentId: -1,
                          accountBooksId: this.defaultAccountBook,
                          subjectCode: this.subjectCodeAdd,
                      })
            if (data.ret === 1) {
                this.$message.success(data.msg)
                this.getList()
            } else {
                this.$message.error(data.msg)
            }
        },
        async updateProject() {
            let { data } = await update({
                ...this.form,
                assistAccount: this.assistAccountStr,
                accountBooksId: this.defaultAccountBook,
                subjectCode: this.subjectCodeEdit,
                fullName: this.fullName,
            })
            if (data.ret === 1) {
                this.form = {
                    subjectCode: '',
                    subjectName: '',
                    subjectTypeId: '',
                    balanceDirect: '',
                    status: '',
                    cashSubject: '',
                    bankSubject: '',
                    foreiCurrAcc: '',
                    endRemittance: '',
                    contactsBusiness: '',
                }
                this.$message.success(data.msg)
                this.getList()
            } else {
                this.$message.error(data.msg)
            }
        },

        async deleteProject(type, row) {
            let { data } =
                type === 'one'
                    ? await deleteBy({
                          ids: [row.id],
                      })
                    : await deleteBy({
                          ids: this.selList,
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
                    ? `确定删除科目名称为 ${row.subjectName} 的数据吗？`
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
            this.selList = rows.map(item => item.id)
            // console.log(this.selList)
        },
        handleSelectAll() {
            const isAllSelected = this.$refs.multipleTable.store.states.isAllSelected
            let _handleSelectAll = data => {
                data.forEach(item => {
                    this.$refs.multipleTable.toggleRowSelection(item, isAllSelected)
                    _handleSelectAll(item.children || [])
                })
            }
            _handleSelectAll(this.tableData.list)
        },
        handleSelect(selection, current) {
            // console.log(selection, current)
            // 判断selection中是否存在current,若是存在那么就代表是被勾选上了,若是不存在代表是取消勾选了
            const isChecked = !!selection.find(item => item.id === current.id)
            // 如果当前项被取消勾选
            if (!isChecked) {
                // 那么其所有的祖先也应该被取消勾选
                this.uncheckedParents(selection, current)
                // 那么其所有的后代也应该被取消勾选
                this.toggleCheckedChildrens(selection, current, false)
            } else {
                // 如果当前项被勾选
                // 那么若同一组的元素都被勾选了,那么父元素将也被勾选,依次往上类推
                this.checkedParents(selection)
                // 那么其所有的后代都要被勾选
                this.toggleCheckedChildrens(selection, current, true)
            }
        },
        uncheckedParents(selection, item) {
            let _uncheckedParents = data => {
                return data.find(element => {
                    if (element.id === item.id) {
                        return true
                    } else if (_uncheckedParents(element.children || [])) {
                        this.$refs.multipleTable.toggleRowSelection(element, false)
                        for (let i = selection.length - 1; i >= 0; i--) {
                            if (selection[i].id === element.id) {
                                selection.splice(i, 1)
                                break
                            }
                        }
                        return true
                    } else {
                        return false
                    }
                })
            }
            _uncheckedParents(this.tableData.list)
        },
        toggleCheckedChildrens(selection, item, isChecked) {
            let _toggleCheckedChildrens = data => {
                data.find(element => {
                    this.$refs.multipleTable.toggleRowSelection(element, isChecked)
                    if (isChecked && !selection.find(item => item.id === element.id)) {
                        selection.push(element)
                    } else if (!isChecked && selection.find(item => item.id === element.id)) {
                        for (let i = selection.length - 1; i >= 0; i--) {
                            if (selection[i].id === element.id) {
                                selection.splice(i, 1)
                                break
                            }
                        }
                    }
                    _toggleCheckedChildrens(element.children || [])
                })
            }
            _toggleCheckedChildrens(item.children || [])
        },
        checkedParents(selection) {
            let _checkedParents = element => {
                const children = element.children
                if (children && children.length) {
                    const allChildrenChecked = children.every(child => {
                        return _checkedParents(child)
                    })
                    if (allChildrenChecked) {
                        this.$refs.multipleTable.toggleRowSelection(element, true)
                        if (!selection.find(item => item.id === element.id)) {
                            selection.push(element)
                        }
                    }
                }
                return selection.find(item => item.id === element.id)
            }
            this.tableData.list.forEach(element => {
                _checkedParents(element)
            })
        },

        changeSize(pageSize) {
            this.pageSize = pageSize
            this.pageNumber = 1
            this.getList()
        },
        goPage(pageNum) {
            this.pageNumber = pageNum
            this.getList()
        },
        resetMethod() {
            this.searchForm = {}
            this.getList()
        },
    },
}
</script>
<style lang="less" scoped>
@import url('./tab.less');
/deep/.el-form {
    .form-fl-1,
    .form-fl-2 {
        .form-l {
            float: left;
            width: 45%;
        }
        .form-r {
            float: right;
            width: 45%;
        }
        &::after {
            content: '';
            display: block;
            clear: both;
        }
    }
}
/deep/.cell {
    .el-table__expand-icon {
        display: none;
    }
    .el-table__placeholder {
        display: none;
    }
}
/deep/.el-dialog__body {
    .el-form {
        .subject-code {
            .el-form-item__content {
                margin-top: 6px;
                height: 34px;
                line-height: 32px;
                padding: 0 15px;
                border: 1px solid #dcdfe6;
                border-radius: 4px;
                overflow: hidden;
                .subject-code-fixed {
                    float: left;
                }
                .subject-code-edit {
                    float: left;
                    width: 35px;
                    input {
                        width: 35px;
                        padding: 0;
                        border-color: transparent;
                        font-size: 14px;
                    }
                }
            }
        }
        .diseditActive {
            .el-form-item__content {
                background: rgb(245, 247, 250);
                span {
                    color: #c0c4cc;
                }
            }
        }
    }
}
</style>
