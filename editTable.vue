<template>
    <div id="editTable" @mouseleave="mouseLeave">
        <div class="check-image" v-show="isEdit">
            <img
                :class="isEdit && spriteStyle ? 'spriteStyle' : ''"
                src="@/assets/images/voucher.png"
                alt="凭证精灵图"
            />
        </div>
        <table>
            <thead>
                <tr v-for="(item, index) in rownum" :key="index">
                    <th v-for="(item, index) in thlabel[index]" :key="index">{{ item.label }}</th>
                </tr>
            </thead>
            <tbody>
                <tr
                    v-for="(row, index) in datat"
                    :key="index"
                    @mouseover="mouseOver(index)"
                    @click="clickSelectCell(row, index)"
                >
                    <td
                        v-for="(key, cIndex) in labelprop"
                        :key="cIndex"
                        :class="
                            (key === 'trsamtd' || key === 'trsamtc') && row[key].includes('-') ? 'unique-color' : ''
                        "
                    >
                        <el-input
                            type="textarea"
                            v-model="row[key]"
                            @focus="handleFocusInput"
                            v-if="key === 'brief'"
                            @keydown.native="handleKeyDownCell"
                            :disabled="isEdit"
                        ></el-input>
                        <editSelect
                            v-model="row[key]"
                            :optionList="optionList"
                            :editIndex="index"
                            :showContent.sync="row.showAccountSubject"
                            @getTarget="targetFromEditS"
                            @handle-focus="handleFocus"
                            v-else-if="key === 'accountSubject'"
                            @keydown.native="handleKeyDownCell"
                            @showSubAccountDialog="clickSubDialog"
                            :isEdit="isEdit"
                            @handle-trigger="handleTrigger"
                            :initOptionLabelList="initOptionLabelList"
                            @focus="handleTriggleSub(row)"
                        ></editSelect>
                        <!-- :disabled="isEditAccount" -->
                        <el-input
                            type="text"
                            v-model="row[key]"
                            @change="inputCell(row, key, row[key])"
                            @focus="handleFocusInput"
                            v-else
                            @keydown.native="handleKeyDownCell"
                            :disabled="isEdit"
                        ></el-input>
                    </td>
                </tr>
            </tbody>
            <tfoot>
                <tr>
                    <td colspan="2">合计：{{ totalAmountCap }}</td>
                    <td class="text-position">{{ totalTrsamtd !== '0.00' ? totalTrsamtd : '' }}</td>
                    <td class="text-position">{{ totalTrsamtc !== '0.00' ? totalTrsamtc : '' }}</td>
                </tr>
            </tfoot>
        </table>
        <div :class="[isShowIconBtn ? 'iconBtn' : 'hiddenIconBtn']" ref="iconBtn" v-if="!isEdit">
            <!-- <i class="el-icon-circle-plus addIcon" @click="addRow"></i> -->
            <i class="el-icon-caret-top addIcon" @click="addRow"></i>
            <i class="el-icon-error delateIcon" @click="delateRow"></i>
            <i class="el-icon-caret-bottom addIcon" @click="addRowDown"></i>
        </div>
        <div class="cashierInfo">
            <span>制单人: {{ cashierInfo.createZh ? cashierInfo.createZh : username }}</span>
            <span v-if="cashierInfo.checkZh">审核人: {{ cashierInfo.checkZh }}</span>
            <span v-if="cashierInfo.posterZh">过账人: {{ cashierInfo.posterZh }}</span>
            <span v-if="cashierInfo.cashierZh">出纳人: {{ cashierInfo.cashierZh }}</span>
        </div>
        <el-dialog
            title="辅助核算"
            :visible.sync="subVisible"
            width="35%"
            :show-close="false"
            :close-on-click-modal="false"
            :close-on-press-escape="false"
        >
            <!-- :rules="rules" :show-message="false" -->
            <el-form label-width="100px" ref="ValidateForm" :model="ValidateForm" size="mini">
                <div v-for="(item, index) in ValidateForm.subList" :key="index">
                    <template v-if="item">
                        <el-form-item
                            :label="item.name"
                            :prop="'subList.' + index + '.subModelID'"
                            :rules="{ required: true, message: `${item.name} 不能为空` }"
                        >
                            <el-select v-model="item.subModelID" @change="changeSubOption(item)" filterable>
                                <el-option
                                    v-for="(subItem, subIndex) in item.subInfoList"
                                    :key="subIndex"
                                    :value="subItem.userId || subItem.departId || subItem.companyId || subItem.id"
                                    :label="subItem.initSubLabel"
                                ></el-option>
                            </el-select>
                        </el-form-item>
                    </template>
                </div>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click="cancelMethod" size="mini" w-65>取 消</el-button>
                <el-button type="primary" @click="cofirmMethod('ValidateForm')" size="mini" w-65>确 定</el-button>
            </div>
        </el-dialog>
    </div>
</template>

<script>
import editSelect from '@/components/voucher/editSelect'
import { parseFormatNum } from '@/utils/index'
import { findAllSubject, findOtherSub, findAllDepart, findAllEmployer, findAllSupplier } from '@/api/fims/voucherSelect'
export default {
    name: 'editTable',
    props: {
        thlabel: {
            //表头数组
            type: Array,
            required: true,
        },
        datat: {
            type: Array,
            required: true,
        },
        isEdit: {
            type: Boolean,
            required: true,
        },
        spriteStyle: {
            type: Boolean,
            required: true,
        },
        isEditAccount: {
            type: Boolean,
            required: true,
        },
        totalAmountCap: {
            type: String,
            required: true,
        },
        totalTrsamtd: {
            type: String,
            required: true,
        },
        totalTrsamtc: {
            type: String,
            required: true,
        },
        disabledBtn: {
            type: Object,
            required: true,
        },
        cashierInfo: {
            type: Object,
            required: true,
        },
        accountBooksId: {
            type: Number,
            required: true,
        },
    },
    model: {
        prop: 'disabledBtn',
        event: 'handle-keydown-cell',
    },
    components: { editSelect },
    data() {
        return {
            isShowIconBtn: false,
            index: 0,
            optionList: [],
            vSelectModel: '',
            subjectVisible: false,
            subVisible: false,
            placeholder: '',
            // subInfoList: [],
            cellIndex: 0,
            assistLabel: '',
            assistAccountIds: '',
            username: '',
            ckdArray: [],
            // rules: {
            //     subModelID: [{ required: true, trigger: 'change' }],
            // },
            ValidateForm: {
                subList: [],
            },
            initOptionList: [],
            initOptionLabelList: [],
            assActIdsList: [],
            subOthers: false,
        }
    },
    computed: {
        //表头行数
        rownum() {
            return this.thlabel.length
        },
        //取出表头数据依次对应的字段key
        labelprop() {
            let thlab = this.thlabel
            let arr = []
            for (let i = 0; i < thlab.length; i++) {
                for (let j = 0; j < thlab[i].length; j++) {
                    for (let key in thlab[i][j]) {
                        if (key === 'prop') {
                            arr.push(thlab[i][j][key])
                        }
                    }
                }
            }
            // console.log(arr)
            return arr
        },
        changeIsInitEnter() {
            return this.$store.state.isInitEnter
        },
    },
    created() {
        // console.log(this.datat, this.totalAmountCap)
        let { username } = JSON.parse(localStorage.getItem('userInfo'))
        this.username = username
        // console.log('$refs.editTable：' + this.changeIsInitEnter)
        this.$nextTick(() => {
            if (this.changeIsInitEnter) {
                this.getAllSubject()
            }
        })
    },
    methods: {
        // 会计科目信息
        async getAllSubject(value) {
            // console.log('帐套ID：' + value)
            let newValue = value ? value : this.accountBooksId
            let { data } = await findAllSubject({
                pageNo: 1,
                pageSize: 1000,
                accountBooksId: newValue,
            })
            this.optionList = data.data.list
            this.optionList.map(v => {
                let key = 'initLabel'
                // let value = `${v.subjectCode} ${v.subjectName}`
                let value = `${v.subjectCode} ${v.fullName}`
                return (v[key] = value)
            })
            this.initOptionList = JSON.parse(JSON.stringify(this.optionList))
            this.initOptionLabelList = this.initOptionList.map(v => v.initLabel)
            // console.log(this.optionList.length, this.initOptionLabelList.length)
        },

        inputCell(row, key, val) {
            // console.log(row, key, val)
            // 校验
            let newVal = val + ''
            /* !/^(([1-9]\d*)|\d)(\.\d{1,2})?$/  !/^(\-|\+)?\d+(\.\d+)?$/ */
            // /^(0|-?[1-9][0-9]*)(.[0-9]{1,2})?$/ -0.开头的小数校验不通过
            if (!/^([\+ \-]?(([1-9]\d*)|(0)))([.]\d{0,2})?$/.test(newVal)) {
                this.$message.error('金额只能为数字,且最多保留两位小数')
                key === 'trsamtc' ? (row.trsamtc = '') : (row.trsamtd = '')
            } else {
                key === 'trsamtc'
                    ? (row.trsamtc = parseFormatNum(Number(val), 2))
                    : (row.trsamtd = parseFormatNum(Number(val), 2))
                // console.log(parseFormatNum(Number(val), 2))
            }
            // 互斥
            if (key === 'trsamtc') {
                row.trsamtd = ''
            } else if (key === 'trsamtd') {
                row.trsamtc = ''
            }
        },

        mouseOver(index) {
            this.index = index
            this.isShowIconBtn = true
            if (!this.isEdit) {
                this.$refs.iconBtn.style.top = `${index * 51 + 65}px`
            }
        },
        mouseLeave() {
            this.isShowIconBtn = false
        },
        addRow() {
            let addEmptyObj = {
                brief: '',
                accountSubject: '',
                trsamtd: '',
                trsamtc: '',
                subjectId: '',
                assistAccountIds: '',
                showAccountSubject: false,
            }
            this.datat.splice(this.index, 0, addEmptyObj)
            this.handleKeyDownCell()
            // this.datat.push(addEmptyObj)
        },
        addRowDown() {
            let addEmptyObj = {
                brief: '',
                accountSubject: '',
                trsamtd: '',
                trsamtc: '',
                subjectId: '',
                assistAccountIds: '',
                showAccountSubject: false,
            }
            this.datat.splice(this.index + 1, 0, addEmptyObj)
            this.handleKeyDownCell()
        },
        delateRow() {
            this.datat.splice(this.index, 1)
            this.isShowIconBtn = false
            this.handleKeyDownCell()
        },

        // 模糊搜索
        targetFromEditS(target) {
            // console.log(target)
            let newOptionList = []
            if (target) {
                this.initOptionList.filter(item => {
                    if (item.initLabel.indexOf(target) !== -1) {
                        newOptionList.push(item)
                    }
                })
                this.optionList = newOptionList
            } else {
                this.optionList = this.initOptionList
            }
            // console.log(this.optionList.length)
        },

        // 辅助科目弹窗
        clickSubDialog(ckdSelect) {
            this.ckdArray = []
            // console.log('辅助核算id: ' + this.datat[this.cellIndex].assistAccountIds)
            if (ckdSelect.assistAccountInfo) {
                // 整合数据格式
                // console.log(ckdSelect.assistAccountInfo)
                ckdSelect.assistAccountInfo.forEach(async (v, index) => {
                    this.getSubInfo(v.id).then(res => {
                        v.subInfoList = res
                        if (this.assActIdsList.length !== 0) {
                            this.$set(this.ValidateForm.subList, index, { ...v, subModelID: this.assActIdsList[index] })
                            // console.log(this.assActIdsList)
                            this.changeSubOption(this.ValidateForm.subList[index])
                            this.subOthers = true
                        } else {
                            this.$set(this.ValidateForm.subList, index, { ...v, subModelID: '' })
                            this.subOthers = false
                        }
                        if (index === ckdSelect.assistAccountInfo.length - 1) {
                            // console.log('辅助核算弹窗...')
                            this.subVisible = true
                        }
                    })
                })
                this.assistLabel = ckdSelect.initLabel
                // console.log('选择元素初始assistLabel: ' + this.assistLabel, '所在行：' + this.cellIndex)
            }
        },
        async getSubInfo(type) {
            let temp_data
            let { data } =
                type === 2
                    ? await findAllDepart({
                          pageNum: 1,
                          pageSize: 1000,
                      })
                    : type === 3
                    ? await findAllEmployer({
                          pageNum: 1,
                          pageSize: 1000,
                      })
                    : type === 4
                    ? await findAllSupplier({
                          pageNum: 1,
                          pageSize: 5000,
                      })
                    : await findOtherSub({
                          pageNum: 1,
                          pageSize: 1000,
                          assisTypeId: type,
                      })
            if (data.ret === 1) {
                temp_data = data.data.list.map(v => {
                    let key = 'initSubLabel'
                    let value
                    if (type === 2) {
                        value = `[${v.assisTypeCode}]${v.departName}`
                    } else if (type === 3) {
                        value = `[${v.assisTypeCode}]${v.userNameZh}`
                    } else if (type === 4) {
                        value = `[${v.assisTypeCode}]${v.compayName}`
                    } else {
                        value = `[${v.assisTypeCode}]${v.assisTypeName}`
                    }
                    v[key] = value
                    return v
                })
            }
            return temp_data
        },
        clickSelectCell(row, index) {
            // console.log(row, row.accountSubject)
            // 点击的哪一行的select?
            this.cellIndex = index
            // 选中的subjectID?
            let ckdSubject = this.initOptionList.filter(v => {
                if (v.initLabel === row.accountSubject) {
                    return v
                }
            })
            // console.log(ckdSubject[0])
            // 科目id值
            if (ckdSubject[0]) {
                row.subjectId = ckdSubject[0].id
            }
            // console.log(this.datat)
        },

        getSubjectLabel(val, id, cid, curAccountSubject) {
            let ckdObj = {
                val: val,
                id: id,
                cid: cid,
            }
            if (this.ckdArray.length > 0) {
                this.ckdArray.filter((v, index) => {
                    if (v.id === id) {
                        this.ckdArray.splice(index, 1)
                    }
                })
            }
            this.ckdArray.push(ckdObj)
            let label = this.ckdArray.map(v => v.val).join('/')
            this.assistLabel = `${curAccountSubject}/${label}`
            this.assistAccountIds = this.ckdArray.map(v => `${v.id}_${v.cid}`).join(',')
            // console.log(this.ckdArray, this.assistLabel, this.assistAccountIds)
        },
        changeSubOption(item) {
            // console.log('辅助核算： ', item)
            let id = item.id
            let cid = item.subModelID
            // console.log(item.subInfoList)
            let ckdEle = item.subInfoList.filter(v => {
                if (
                    v.userId === item.subModelID ||
                    v.departId === item.subModelID ||
                    v.companyId === item.subModelID ||
                    v.id === item.subModelID
                ) {
                    return v
                }
            })
            // console.log(ckdEle[0])
            let val = ckdEle[0].initSubLabel
            let curAccountSubject = this.datat[this.cellIndex].accountSubject
            // console.log(this.assActIdsList)
            if (this.assActIdsList.length !== 0) {
                curAccountSubject = curAccountSubject.substring(0, curAccountSubject.indexOf('/'))
            }
            // console.log(val, id, cid, curAccountSubject)
            this.getSubjectLabel(val, id, cid, curAccountSubject)
        },
        // 辅助核算 -> 动态校验
        cofirmMethod(formName) {
            this.$refs[formName].validate(valid => {
                if (valid) {
                    this.subVisible = false
                    this.ValidateForm.subList.map(v => (v.subModelID = ''))
                    this.datat[this.cellIndex].accountSubject = this.assistLabel
                    this.datat[this.cellIndex].assistAccountIds = this.assistAccountIds
                    this.ValidateForm.subList = []
                    this.ckdArray = []
                    this.assActIdsList = []
                }
            })
        },
        cancelMethod() {
            // console.log(this.subOthers)
            this.subVisible = false
            this.ValidateForm.subList = []
            this.ckdArray = []
            this.assActIdsList = []
            if (!this.subOthers) {
                this.datat[this.cellIndex].accountSubject = ''
                this.datat[this.cellIndex].assistAccountIds = ''
            }
            if (this.datat[this.cellIndex].accountSubject === '') {
                this.datat[this.cellIndex].assistAccountIds = ''
            }
            // console.log(this.datat[this.cellIndex].assistAccountIds, this.datat[this.cellIndex].accountSubject)
        },

        TriggleSelectCell(row) {
            // console.log(row, row.assistAccountIds, row.subjectId)
            let subjectId = row.subjectId
            this.assActIdsList = []
            if (row.assistAccountIds) {
                let assActIds = row.assistAccountIds.split(',')
                let assActIdsList = []
                assActIds.forEach(item => {
                    let assid = item.substring(item.lastIndexOf('_') + 1)
                    assid = Number(assid)
                    assActIdsList.push(assid)
                })
                // console.log(assActIds, assActIdsList)
                this.assActIdsList = assActIdsList
                let ckdSelectSubject = null
                this.initOptionList.forEach(item => {
                    if (subjectId === item.id) {
                        ckdSelectSubject = item
                    }
                })
                // console.log(ckdSelectSubject)
                if (ckdSelectSubject) {
                    if (ckdSelectSubject.assistAccountInfo) {
                        // console.log('弹窗...')
                        this.clickSubDialog(ckdSelectSubject)
                    } else {
                        this.$message.error('数据异常，该数据不存在辅助核算项目')
                        row.showAccountSubject = true
                    }
                } else {
                    this.$message.error('数据异常，无法修改该数据的辅助核算项目')
                    row.showAccountSubject = true
                }
            }
        },
        // 有且仅能打开一个select菜单
        handleFocus(editIndex) {
            // console.log(this.datat[editIndex], editIndex)
            this.cellIndex = editIndex
            let selectedRow = this.datat[editIndex]
            this.datat.forEach((item, index) => {
                if (editIndex === index) {
                    // console.log('editIndex' + editIndex, 'index' + index)
                    if (selectedRow.assistAccountIds) {
                        item.showAccountSubject = false
                        this.TriggleSelectCell(selectedRow)
                    } else {
                        item.showAccountSubject = true
                    }
                } else {
                    item.showAccountSubject = false
                }
            })
            this.$forceUpdate()
        },
        handleFocusInput() {
            this.$emit('handle-focus-input')
        },

        // 触发表格  点击下拉菜单元素未触发
        handleKeyDownCell() {
            // console.log(false)
            let disabledBtn = {
                disabledAdd: false,
                disabledSava: false,
                disabledTemSava: false,
                disabledCheck: true,
            }
            this.$emit('handle-keydown-cell', disabledBtn)
        },
        handleTrigger() {
            this.handleKeyDownCell()
            // 清空上一次的辅助核算记录
            // console.log('清除辅助核算id', this.datat[this.cellIndex].assistAccountIds)
            this.datat[this.cellIndex].assistAccountIds = ''
        },
    },
}
</script>

<style lang="less" scoped>
#editTable {
    position: relative;
    width: 100%;
    // margin: 0 40px;
    padding: 0 40px;
    box-sizing: border-box;
    .check-image {
        position: absolute;
        top: -50px;
        left: 65%;
        z-index: 99;
        width: 130px;
        height: 65px;
        overflow: hidden;
        img {
            display: block;
            margin-top: -40px;
            margin-left: 10px;
        }
        .spriteStyle {
            margin-left: -120px;
        }
    }
    table {
        border: 1px solid #ebeef5;
        // width: calc(100% - 80px);
        width: 100%;
        thead {
            tr {
                th {
                    border: 1px solid #dadada;
                    color: #606266;
                    line-height: 55px;
                    text-align: center;
                }
            }
        }
        tbody {
            tr {
                td {
                    position: relative;
                    height: 50px;
                    border: 1px solid #dadada;
                    color: #606266;
                    /deep/.el-textarea,
                    /deep/.el-input {
                        display: block;
                        width: 100%;
                        text-align: left;
                        textarea,
                        input {
                            height: 50px;
                            padding: 0 5px;
                            border-radius: 0;
                            border: none;
                            outline: none;
                            resize: none;
                            &:focus {
                                border: 1px solid #0079fe;
                            }
                        }
                        input {
                            text-align: right;
                        }
                    }
                }
                .unique-color {
                    /deep/.el-input {
                        input {
                            color: #ff0046;
                        }
                    }
                }
            }
        }
        tfoot {
            tr {
                td {
                    height: 50px;
                    border: 1px solid #dadada;
                    color: #606266;
                    padding: 0 5px;
                }
                .text-position {
                    text-align: right;
                }
            }
        }
    }
    .iconBtn {
        position: absolute;
        top: 0px;
        left: 20px;
        i {
            display: block;
            color: #0079fe;
            &:nth-last-child(2) {
                // margin-top: 5px;
                color: #ff0000;
            }
        }
    }
    .hiddenIconBtn {
        display: none;
    }
    .cashierInfo {
        margin-top: 10px;
        span {
            display: inline-block;
            margin-right: 20px;
        }
    }
    /deep/.el-dialog {
        .el-form-item {
            width: 100%;
            .el-select {
                width: 100%;
                .el-input {
                    width: 100%;
                    input {
                        width: 100%;
                    }
                }
            }
        }
    }
}
</style>
