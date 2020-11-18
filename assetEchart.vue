<template>
    <div class="jexcel">
        <div class="search-btn" ref="searchBox">
            <el-form :model="searchForm" inline ref="searchForm">
                <el-row>
                    <el-form-item>
                        <el-select
                            v-model.number="searchForm.accountBooksId"
                            placeholder="帐套"
                            class="filter-item"
                            size="mini"
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
                        <el-form-item>
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
                    <el-form-item>
                        <el-select
                            v-model.trim="searchForm.tableType"
                            placeholder="报表类型"
                            class="filter-item"
                            size="mini"
                            clearable
                            filterable
                            @change="getTableId"
                        >
                            <el-option
                                v-for="item in echartTypeList"
                                :key="item.id"
                                :label="item.tableValue"
                                :value="item.tableType"
                            ></el-option>
                        </el-select>
                    </el-form-item>
                </el-row>
                <div class="btn-list">
                    <el-button @click="getEchart()" :disabled="showSearch" type="primary" size="mini" w-90
                        >查询</el-button
                    >
                    <el-button @click="resetMethod" size="mini" w-90>重置</el-button>
                </div>
            </el-form>
        </div>
        <div class="table-box" ref="tableBox">
            <div class="spread-btn">
                <!-- <el-button @click="getMergeCell()" size="mini">获取合并的单元格</el-button> -->
                <!-- <el-button @click="setInitCell('')" size="mini">设置数据(测试)</el-button> -->
                <el-button @click="getCellValue('add')" size="mini">新增报表</el-button>
                <el-button @click="getCellValue('save')" size="mini" :disabled="showFormula">保存报表</el-button>
                <el-button @click="getCellValue('change')" size="mini">{{
                    showFormula ? '展示公式' : '展示值'
                }}</el-button>
            </div>
            <div id="spread" ref="spread"></div>
        </div>
        <el-dialog
            :title="id ? '保存报表' : '新增报表'"
            :visible.sync="dialogVisible"
            width="30%"
            @close="cancelMethod"
        >
            <el-form :model="form" label-width="80px" :rules="formRule" ref="form">
                <el-form-item label="报表名称" prop="tableValue" v-if="id">
                    <el-select
                        v-model.trim="searchForm.tableType"
                        placeholder="请选择报表名称"
                        class="filter-item"
                        size="mini"
                        filterable
                        @change="getTableId"
                    >
                        <el-option
                            v-for="item in echartTypeList"
                            :key="item.id"
                            :label="item.tableValue"
                            :value="item.tableType"
                        ></el-option>
                    </el-select>
                </el-form-item>
                <el-form-item label="报表名称" prop="tableValue" v-else>
                    <el-input
                        v-model.trim="form.tableValue"
                        placeholder="请输入报表名称"
                        clearable
                        size="mini"
                    ></el-input>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click="cancelMethod" size="mini" w-65>取 消</el-button>
                <el-button type="primary" @click="cofirmMethod" size="mini" w-65>确 定</el-button>
            </div>
        </el-dialog>
    </div>
</template>

<script>
import jexcelStyle from 'jexcel/dist/jexcel.css' // eslint-disable-line no-unused-vars
import jexcel from 'jexcel' // eslint-disable-line no-unused-vars
import jSuites from 'jSuites'
import { deepCopy, dealTotastMsg, parseFormatNum } from '@/utils/index'
import { findAllAccountBook, findVouDur, findEchart, findEchartType, add, updata } from '@/api/fims/echart.js'

let mergeStart = null
let toolbar = [
    {
        type: 'i',
        content: 'undo',
        onclick: function(element, instance) {
            instance.undo()
        },
    },
    {
        type: 'i',
        content: 'redo',
        onclick: function(element, instance) {
            instance.redo()
        },
    },
    {
        type: 'i',
        content: 'cloud_download',
        onclick: function(element, instance) {
            instance.download()
        },
    },
    {
        type: 'select',
        k: 'font-family',
        v: ['Arial', 'Verdana'],
    },
    {
        type: 'select',
        k: 'font-size',
        v: ['9px', '10px', '11px', '12px', '13px', '14px', '15px', '16px', '17px', '18px', '19px', '20px'],
    },
    {
        type: 'i',
        content: 'format_align_left',
        k: 'text-align',
        v: 'left',
    },
    {
        type: 'i',
        content: 'format_align_center',
        k: 'text-align',
        v: 'center',
    },
    {
        type: 'i',
        content: 'format_align_right',
        k: 'text-align',
        v: 'right',
    },
    {
        type: 'i',
        content: 'tab_unselected',
        onclick: function(element, instance) {
            // console.log(instance, instance.selectedCell, mergeStart)
            let mergeRows = Number(instance.selectedCell[3]) - Number(instance.selectedCell[1]) + 1
            let mergeCols = Number(instance.selectedCell[2]) - Number(instance.selectedCell[0]) + 1
            // console.log(mergeCols, mergeRows)
            instance.setMerge(mergeStart, mergeCols, mergeRows) // 起始单元格名称, 合并列, 合并行
        },
    },
    {
        type: 'i',
        content: 'format_bold',
        k: 'font-weight',
        v: 'bold',
    },
    {
        type: 'i',
        content: 'format_color_text',
        k: 'color',
        onclick: function(element, instance, item) {
            if (!item.color) {
                // console.log(jSuites.color)
                let colorPicker = jSuites.color(item, {
                    onchange: function(o, v) {
                        cellPicker(o, v, instance, 'color')
                        // console.log(instance, instance.selectedCell, o, v, element)
                    },
                })
                colorPicker.open()
            }
        },
    },
    {
        type: 'color',
        content: 'format_color_fill',
        k: 'background-color',
        onclick: function(element, instance, item) {
            if (!item.color) {
                let colorPicker = jSuites.color(item, {
                    onchange: function(o, v) {
                        cellPicker(o, v, instance, 'backgroundColor')
                    },
                })
                colorPicker.open()
            }
        },
    },
]

// 字体颜色/背景色
let cellPicker = function(o, v, instance, style) {
    let color = v,
        // 选中单元格classname highlight, instance.selectedCell选中单元格[y, x, mergeY, mergeX]
        fouseCell = instance.selectedCell,
        y = Number(fouseCell[0]),
        x = Number(fouseCell[1]),
        y2 = Number(fouseCell[2]),
        x2 = Number(fouseCell[3])
    for (let i = x; i <= x2; i++) {
        for (let j = y; j <= y2; j++) {
            let scell = instance.getCell([j, i])
            // console.log(scell)
            scell.style.cssText += `${style}: ${color}`
        }
    }
}

let text = {
    insertANewColumnBefore: '往左增加一列',
    insertANewColumnAfter: '往右增加一列',
    deleteSelectedColumns: '删除选中列',
    insertANewRowBefore: '往前增加一行',
    insertANewRowAfter: '往后增加一行',
    deleteSelectedRows: '删除选中行',
    renameThisColumn: '重置列名',
    orderAscending: '升序',
    orderDescending: '降序',
    copy: '复制',
    paste: '粘贴',
    saveAs: '保存为',
    about: '关于',
}

export default {
    name: 'assetEchart',
    data() {
        return {
            id: null, // 区分新增or修改报表
            btnType: null, // 选中按钮
            searchForm: {},
            accountBookList: [],
            vouDurList: [],
            echartTypeList: [],
            spread: null,
            sheet: null,
            showFormula: false, // true: 展示公式，当前看到的是值； false: 展示值，当前看到的是公式
            temporaryData: null,
            dialogVisible: false,
            unemptyCell: null,
            unemptyCellLen: 0,
            minDimensions: [26, 28], // 表格列数，行数[colslen, rowslen]
            cell: null,
            // mergeStart: null, // 合并单元格的起始行
            mergeCellAll: null,
            form: {},
            formRule: {
                tableValue: [
                    {
                        required: true,
                        message: '请输入表单名称',
                        trigger: 'blur',
                    },
                ],
            },
            preValue: '',
            changeCell: '',
            initSheet: [
                {
                    location: '2,2',
                    formula: '=ACCT(1001:1002,Y,1,4,4)',
                },
                {
                    location: '3,2',
                    formula: '=ACCT(1121:1122,Y,1,4,4)+ACCT(1131,Y,1,4,4)',
                },
                {
                    location: '4,2',
                    formula: '=ACCT(1221,Y,1,4,4)',
                },
                {
                    location: '5,2',
                    formula: '=ACCT(2221,Y,1,4,4)',
                },
                {
                    location: '7,2',
                    formula: '=ACCT(6001,Y,1,4,4)',
                },
                {
                    location: '10,2',
                    formula: '=SUM(C2:C3)+SUM(C5:C10)',
                },
            ],
        }
    },
    computed: {
        showSearch() {
            if (
                this.searchForm.accountBooksId &&
                this.searchForm.startPeriodId &&
                this.searchForm.endPeriodId &&
                this.searchForm.tableType
            ) {
                return false
            } else {
                return true
            }
        },
    },
    mounted() {
        this.getAllAccountBook()
        this.getEchartType()
        this.createJExcel()
        this.getInitHeight()
    },
    methods: {
        setInitCell(temporaryData) {
            // console.log(temporaryData)
            let data = temporaryData ? temporaryData : this.initSheet
            data.forEach(v => {
                if (v) {
                    let locX = v.location.split(',')[0] * 1
                    let locY = v.location.split(',')[1] * 1
                    this.spread.setValue(this.spread.getCell([locY, locX]), v.formula)
                    // 展示公式情况下，=号公式不计算
                    if (!this.showFormula) {
                        this.spread.getCell([locY, locX]).innerText = v.formula
                        this.spread.getCell([locY, locX]).style.textAlign = 'center'
                        // if (v.formula.includes('=')) {  // 只有公式才需要重新设置样式居中
                        //     this.spread.getCell([locY, locX]).style.textAlign = 'center'
                        // }
                    }
                }
            })
        },
        // 页面高度计算
        getInitHeight() {
            window.addEventListener('resize', this.$calcTableHeight())
        },

        // 报表类型
        async getEchartType() {
            let { data } = await findEchartType({
                pageNo: 1,
                pageSize: 1000,
            })
            if (data.ret === 1) {
                this.echartTypeList = data.data.list
                // console.log(this.echartTypeList)
            }
        },

        // 帐套
        async getAllAccountBook() {
            let { data } = await findAllAccountBook({
                pageNo: 1,
                pageSize: 200,
            })
            if (data.ret === 1) {
                this.accountBookList = data.data.list
                this.$set(this.searchForm, 'accountBooksId', this.accountBookList[0].accountBooksId)
                this.getVouDur()
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
                this.vouDurList = data.data.list
                // console.log(this.vouDurList)
                // this.vouDurList.forEach(item => {
                //     if (item.isCurrentPeriod === 1) {
                //         this.searchForm.startPeriodId = item.periodId
                //         this.searchForm.endPeriodId = item.periodId
                //     }
                // })
                // this.getList()
            }
        },

        // 生成jexcel
        createJExcel() {
            // console.log(this.$refs['spread'])
            let option = {
                data: [[]],
                minDimensions: this.minDimensions,
                allowToolbar: true,
                toolbar: toolbar,
                text: text,
                onbeforechange: (instance, cell, x, y, value) => {
                    if (!this.showFormula) {
                        this.preValue = value
                        this.changeCell = cell
                    }
                },
                onchange: () => {
                    if (!this.showFormula) {
                        // 展示公式情况下，=号公式不计算
                        this.changeCell.innerText = this.preValue
                    }
                },
                onselection: (instance, x1, y1, x2, y2) => {
                    let cellName1 = jexcel.getColumnNameFromId([x1, y1])
                    let cellName2 = jexcel.getColumnNameFromId([x2, y2])
                    mergeStart = cellName1
                    // console.log(cellName1, cellName2, mergeStart)
                },
            }
            // this.options = option
            this.spread = jexcel(this.$refs['spread'], option)
            // console.log(this.spread)
            Object.assign(this, this.spread)
            // console.log(this.options)
            let sheet = this.options.data
            // console.log(this.options.contextMenu(el, x, y, e))
            this.sheet = deepCopy(sheet)
            // console.log(this.sheet)
        },

        // 查询报表
        async getEchart() {
            let { data } = await findEchart({
                ...this.searchForm,
            })
            if (data.ret === 1) {
                // 清空前一次的表格数据
                this.sheetInit()
                // console.log(data.data)
                let sheetAll = data.data
                let sheetStructure = JSON.parse(data.data.tableStructure)
                let reportData = sheetStructure.reportData
                let cellData = sheetStructure.structure
                // console.log(sheetAll, sheetStructure)
                let mergeData = sheetStructure.mergeCellAll
                // console.log(mergeData)
                if (reportData.length > 0) {
                    // console.log('查询...', sheetStructure)
                    this.id = sheetAll.id
                    // 兼容之前的
                    if (!cellData.length && reportData) {
                        // console.log('兼容之前样式...')
                        this.setCellValue(reportData)
                    }
                    // 包含样式文本与表格行列总数
                    if (sheetStructure.minDimensions) {
                        // console.log('增加样式与表格行列总数...')
                        let minDimensions = sheetStructure.minDimensions
                        // this.spread.options.minDimensions = sheetStructure.minDimensions
                        let lackX = minDimensions[1] - this.minDimensions[1]
                        let lackY = minDimensions[0] - this.minDimensions[0]
                        // console.log(minDimensions, this.minDimensions, lackX, lackY)
                        for (let i = 0; i < lackX; i++) {
                            this.spread.insertRow()
                        }
                        for (let j = 0; j < lackY; j++) {
                            this.spread.insertColumn()
                        }
                        this.minDimensions = minDimensions
                        this.setCellValue(cellData)
                    }
                    this.btnType = null
                    // 合并单元格
                    if (mergeData) {
                        for (let key in mergeData) {
                            this.spread.setMerge(key, mergeData[key][0], mergeData[key][1])
                        }
                    }
                } else {
                    this.$message.warning('查询报表不存在')
                }
            } else {
                this.$message.error(data.msg)
            }
        },

        // 渲染表格  单元格坐标 -> y,x(A1)
        setCellValue(sheet) {
            sheet.forEach(v => {
                if (v) {
                    let locX = v.location.split(',')[0] * 1
                    let locY = v.location.split(',')[1] * 1
                    // this.spread.setValue(this.spread.getCell([locY, locX]), v.formula)
                    if (v.inlineStyle && v.colwidth) {
                        this.spread.getCell([locY, locX]).style.cssText = v.inlineStyle
                        this.spread.setWidth(locY, v.colwidth)
                    }
                    this.notComputed(locY, locX, v)
                }
            })
            // 从接口获取公式 -> 展示值
            this.showFormula = false
            this.changeCellValue(sheet)
        },

        // 获取合并的单元格
        getMergeCell() {
            // this.spread.setMerge(mergeStart, mergeCols, mergeRows)
            this.mergeCellAll = this.spread.getMerge()
            // console.log(this.mergeCellAll)
        },

        // 展示公式情况下，=号公式不计算
        notComputed(locY, locX, ele) {
            if (!this.showFormula) {
                // 展示公式情况下，=号公式不计算
                this.spread.getCell([locY, locX]).innerText = ele.formula
            }
        },

        // 获取单元格（内容/样式/合并单元格）
        getCellValue(type) {
            this.getMergeCell()
            let rowsLen = this.spread.rows.length
            let colsLen = this.spread.colgroup.length
            let cellTxt = []
            let cell = []
            for (let i = 0; i < rowsLen; i++) {
                for (let j = 0; j < colsLen; j++) {
                    if (this.spread.getLabel([j, i])) {
                        let obj = {
                            location: `${i},${j}`,
                            formula: this.spread.getLabel([j, i]),
                        }
                        let cellall = {
                            location: `${i},${j}`,
                            formula: this.spread.getLabel([j, i]),
                            colwidth: this.spread.getWidth()[j],
                            inlineStyle: this.spread.getCell([j, i]).style.cssText,
                        }
                        cellTxt.push(obj)
                        cell.push(cellall)
                    }
                }
            }
            this.minDimensions = [colsLen, rowsLen]
            this.unemptyCell = deepCopy(cellTxt)
            this.unemptyCellLen = cellTxt.length
            this.cell = deepCopy(cell)
            // console.log(this.cell)
            if (type === 'add') {
                this.hintBeforeAdd(cellTxt)
            } else {
                this.useCellValue(type, cellTxt)
            }
        },

        useCellValue(type, newSheet) {
            if (newSheet.length <= 0) {
                this.$message.warning('表格数据不能为空')
                return
            }
            if (!(this.searchForm.startPeriodId && this.searchForm.endPeriodId)) {
                this.$message.warning('请先选择会计期间')
                return
            }
            if (type === 'add') {
                this.btnType = type
                this.dialogVisible = true
            } else if (type === 'save') {
                this.dialogVisible = true
            } else if (type === 'change') {
                this.changeCellValue(newSheet)
            }
        },

        /*
            新增报表
                ->  初始化报表  ->  单元格清空不保存
                ->  初始化报表  ->  初始化前先保存
        */
        hintBeforeAdd(newSheet) {
            if (this.unemptyCellLen <= 0) return
            this.$confirm('是否在新增报表前保存当前报表内容？', '新增报表', {
                distinguishCancelAndClose: true,
                confirmButtonText: '保存',
                cancelButtonText: '放弃保存',
            })
                .then(() => {
                    this.useCellValue('add', newSheet)
                })
                .catch(action => {
                    this.$message({
                        type: 'info',
                        message: action === 'cancel' ? '放弃保存并离开页面' : '停留在当前页面',
                    })
                    this.sheetInit()
                })
        },
        sheetInit() {
            let rowsLen = this.spread.rows.length
            let colsLen = this.spread.colgroup.length
            for (let i = 0; i < rowsLen; i++) {
                for (let j = 0; j < colsLen; j++) {
                    // console.log('横坐标x：' + i, '纵坐标y：' + j)
                    this.spread.setValue(this.spread.getCell([j, i]), '')
                    this.spread.getCell([j, i]).style.cssText = 'text-align: center;'
                }
            }
            this.id = null
            // 销毁所有合并单元格
            this.spread.destroyMerged()
        },

        // 获取报表id，typeValue
        getTableId() {
            let target = this.echartTypeList.filter(item => this.searchForm.tableType === item.tableType)
            // console.log(target)
            if (target.length <= 0) {
                this.id = null
                return
            }
            this.id = target[0].id
            this.tableValue = target[0].tableValue
            let tableType = target[0].tableType
            this.$set(this.searchForm, 'tableType', tableType)
            // console.log(target, this.id, this.tableValue, tableType)
        },

        /*
            确认选择报表(保存)
                新增保存 ->  add
                修改保存(id)  ->   save
        */
        cofirmMethod() {
            if (this.id) {
                this.saveSheet(this.unemptyCell)
                this.dialogVisible = false
            } else {
                let validateArr = [
                    {
                        key: 'tableValue',
                        msg: '请先输入表单名称',
                    },
                ]

                let result = dealTotastMsg(this.form, validateArr, this)
                if (result) return
                this.$refs['form'].validate(valid => {
                    if (valid) {
                        this.saveSheet(this.unemptyCell)
                        this.dialogVisible = false
                    }
                })
            }
        },
        cancelMethod() {
            this.dialogVisible = false
            this.form = {}
        },

        // 保存
        async saveSheet(sheet) {
            // 获取报表tableType -> 切换用
            let tableType = Math.ceil(Math.random() * 100000) + ''
            let tableStructure = {
                minDimensions: this.minDimensions,
                reportData: sheet,
                structure: this.cell,
                mergeCellAll: this.mergeCellAll,
            }
            let commonObj = {
                ...this.searchForm,
                reportData: sheet,
                tableStructure: JSON.stringify(tableStructure),
            }
            let { data } = this.id
                ? await updata({
                      ...commonObj,
                      tableValue: this.tableValue,
                      id: this.id,
                  })
                : await add({
                      ...commonObj,
                      tableValue: this.form.tableValue,
                      tableType: tableType,
                  })
            if (data.ret === 1) {
                this.$message.success(data.msg)
                this.form = {}
                this.getEchartType()
                // this.id ? this.$set(this.searchForm, 'tableType', this.searchForm.tableType) : (this.id = data.data)
                if (this.id) {
                    this.$set(this.searchForm, 'tableType', this.searchForm.tableType)
                } else {
                    this.id = data.data
                    this.$set(this.searchForm, 'tableType', tableType)
                }
                if (this.btnType === 'add') {
                    this.sheetInit()
                    this.$message({
                        type: 'info',
                        message: '保存修改',
                    })
                }
            } else {
                this.$message.error(data.msg)
            }
        },

        // 切换公式/值
        async changeCellValue(newSheet) {
            if (this.showFormula) {
                // true 值 -> 公式
                this.showFormula = false
                this.setInitCell(this.temporaryData)
            } else {
                // false 公式 ->  值  （调接口）
                // 先需要提示选择会计期间！
                if (this.searchForm.startPeriodId && this.searchForm.endPeriodId) {
                    this.getFormula(newSheet)
                    this.showFormula = true
                } else {
                    this.$message.warning('请选选择会计期间')
                    this.showFormula = false
                }
            }
        },

        // 通过公式获取值 => 临时查询
        async getFormula(sheet) {
            // tableStructure: sheet,
            let { data } = await findEchart({
                ...this.searchForm,
                reportData: sheet,
            })
            if (data.ret === 1) {
                // 公式 -> 值
                this.temporaryData = data.data.reportData
                this.temporaryData.forEach(v => {
                    if (v) {
                        let locX = v.location.split(',')[0] * 1
                        let locY = v.location.split(',')[1] * 1
                        v.amount === null
                            ? this.spread.setValue(this.spread.getCell([locY, locX]), v.formula)
                            : this.spread.setValue(this.spread.getCell([locY, locX]), v.amount)
                        // console.log(v)
                        let numData = Number(this.spread.getCell([locY, locX]).innerText)
                        // console.log(this.spread.getCell([locY, locX]).innerText)
                        if ((v.amount !== null || v.formula.includes('=')) && (numData || numData === 0)) {
                            let parseNum = numData.toFixed(2)
                            this.spread.getCell([locY, locX]).innerText = parseFormatNum(parseNum)
                            this.spread.getCell([locY, locX]).style.cssText += 'text-align: right;'
                        }
                    }
                })
            } else {
                this.$message.error(data.msg)
                this.showFormula = true
            }
        },

        resetMethod() {
            this.searchForm = {}
            this.$set(this.searchForm, 'accountBooksId', this.accountBookList[0].accountBooksId)
            this.sheetInit()
        },
    },
}
</script>

<style lang="less" scoped>
.jexcel {
    width: 100%;
    height: auto;
    border-right: none;
    border-bottom: none;
    background-color: transparent;
    // overflow: hidden;
    .search-btn {
        width: 100%;
        background: #ffffff;
        padding: 4px 30px 0px 27px;
        box-sizing: border-box;
        box-shadow: 0px 0px 5px 0px#d9d9d9;
        /deep/ .el-form {
            .el-row {
                display: inline-block;
                .el-form-item {
                    margin-right: 0;
                    .el-input {
                        width: 150px;
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
            /deep/.btn-list {
                float: right;
                margin-top: 5px;
            }
        }
    }
    .table-box {
        margin: 7px;
        padding: 10px 20px 0;
        background: #fff;
        box-shadow: 0px 0px 5px 0px #d9d9d9;
        box-sizing: border-box;
        .spread-btn {
            margin-bottom: 5px;
        }
        /deep/#spread {
            .jexcel_toolbar {
                margin-bottom: 0;
                border-left: 1px solid #ccc;
                .jexcel_toolbar_item {
                    display: block;
                    margin: 0 2px;
                    padding: 0;
                }
            }
        }
    }
    // 调色板
    /deep/#spread {
        // width: 100%;
        .jexcel_toolbar {
            width: 100%;
            .jcolor {
                table {
                    tr {
                        td {
                            width: 10px;
                        }
                    }
                }
            }
        }

        // .jexcel_content {
        // width: 100%;
        // overflow: auto;
        // table.jexcel {
        //     width: 100%;
        // }
        // }
    }
}
</style>
