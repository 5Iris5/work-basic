<!--
 * @Author: Larissa
 * @Date: 2021-12-20 17:20:17
 * @LastEditTime: 2022-01-10 17:37:00
 * @LastEditors: Please set LastEditors
 * @Description: 打开koroFileHeader查看配置 进行设置: https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE
 * @FilePath: \oa_web\src\components\view\umyTable.vue
-->
<template>
    <div class="umy-table">
        <i ref="propsIcon" class="el-icon-document-checked" @click="openColProps"></i>
        <u-table
            ref="uTable"
            border
            size="mini"
            highlight-current-row
            show-body-overflow="tooltip"
            :row-key="rowkey"
            :height="height"
            :row-height="rowHeight"
            :data-changes-scroll-top="false"
            :fixed-columns-roll="true"
            :row-id="rowId"
            use-virtual
            :treeConfig="{
                children: treeChildren,
                expandAll: true,
                indent: 16,
            }"
            :row-class-name="setRowClassName"
            @select="handleSelect"
            @selection-change="selectChange"
            @header-dragend="dragendHeader"
        >
            <u-table-column v-if="showSelection" type="selection" align="center" />
            <u-table-column v-if="showOperate" :label="operateLabel" :width="operateWidth" :fixed="operateFixed">
                <template slot-scope="scope"
                    ><slot name="oprate" :row="scope.row" v-if="!scope.row.hideSelection"></slot
                ></template>
            </u-table-column>
            <u-table-column
                v-for="(item, index) in tableProps"
                :key="isChangeUI ? item.field + generateUUID() : item.field + index"
                :prop="item.field"
                :label="item.label"
                :width="item.width ? item.width : 100"
                :align="textAlignRightList.includes(item.field) ? 'right' : 'left'"
                :fixed="colFixedList.includes(item.field) ? 'left' : false"
                :tree-node="treeNodeList.includes(item.field) ? true : false"
                :render-header="renderHeader"
            >
                <template v-if="item.children.length">
                    <u-table-column
                        v-for="(cItem, cIdx) in item.children"
                        :key="cIdx"
                        :prop="cItem.field"
                        :label="cItem.label"
                        :width="item.width ? item.width : 150"
                        :align="textAlignRightList.includes(cItem.field) ? 'right' : 'left'"
                    >
                        <template slot-scope="scope">
                            <slot
                                name="normal"
                                :row="{ tit: cItem, data: scope.row, prop: cItem.field, $index: scope.$index }"
                            ></slot>
                        </template>
                    </u-table-column>
                </template>
                <template slot-scope="scope">
                    <slot
                        name="normal"
                        :row="{ tit: item, data: scope.row, prop: item.field, $index: scope.$index }"
                    ></slot>
                </template>
            </u-table-column>
        </u-table>
        <Pagination
            v-if="showPagination"
            :pageNumber="pageNumber"
            :pageSize="pageSize"
            :totalCount="tableData.totalCount"
            :totalPage="tableData.totalPage"
            @go-page="goPage"
            @change-size="changeSize"
        ></Pagination>
        <customColumn
            v-if="showCustomColumn"
            ref="customColumn"
            :customColumnVisible="customColumnVisible"
            :originalColumnList="originalColumnList"
            @close-customColumn="closeCustomColumn"
            @submit-customColumn="submitCustomColumn"
        ></customColumn>
        <customSort
            v-show="showCustomSort"
            ref="customSort"
            :sortVisible="sortVisible"
            :flag="flag"
            :tableHeaderList="tableHeaderList"
            @close-customSort="closeCustomSort"
            @submit-customSort="submitCustomSort"
        >
        </customSort>
    </div>
</template>

<script>
    import Pagination from '@/components/pagination'
    import customColumn from '@/components/view/customColumn'
    import customSort from '@/components/view/customSort'
    import { findCustomCulumn, saveCustomCulumn, saveColProp } from '@/api/system/menu'
    import Sortable from 'sortablejs'
    export default {
        name: 'umyTable',
        components: { Pagination, customColumn, customSort },
        props: {
            isChangeUI: {
                type: Boolean,
                default: false,
            },
            height: {
                type: [String, Number],
                default: () => 0,
            },
            rowHeight: {
                type: Number,
                default: 23,
            },
            columnList: {
                type: Array,
                default: () => [],
            },
            textAlignRightList: {
                type: Array,
                default: () => [],
            },
            colFixedList: {
                type: Array,
                default: () => [],
            },
            rowkey: {
                type: String,
                default: '',
            },
            rowId: {
                type: String,
                default: 'id',
            },
            treeChildren: {
                type: String,
                default: 'children',
            },
            treeNodeList: {
                type: Array,
                default: () => [],
            },
            showSelection: {
                type: Boolean,
                default: true,
            },
            showOperate: {
                type: Boolean,
                default: false,
            },
            operateLabel: {
                type: String,
                default: '操作',
            },
            operateWidth: {
                type: Number,
                default: 100,
            },
            operateFixed: {
                type: [Boolean, String],
                default: false,
            },
            tableData: {
                type: Object || Array,
                default: () => {},
            },
            pageSize: {
                type: Number,
                default: 100,
            },
            pageNumber: {
                type: Number,
                default: 1,
            },
            showPagination: {
                type: Boolean,
                default: true,
            },
            showCustomColumn: {
                type: Boolean,
                default: false,
            },
            viewId: { type: Number },
            tableId: { type: Number },
            showCustomSort: {
                type: Boolean,
                default: false,
            },
            flag: {
                type: String,
                default: '',
            },
        },
        computed: {
            tableProps() {
                // this.columnList.filter((v) => v.isCustomize)
                return this.getCheckedCustomColumnIds(this.columnList)
            },
            leftFixedCols() {
                let _num = 0
                if (this.showSelection) _num = 1
                if (this.showOperate) _num = 1
                if (this.showSelection && this.showOperate) _num = 2
                return _num
            },
        },
        watch: {
            showCustomColumn(val) {
                if (val) this.getCustomCulumns()
            },
            customColumnVisible(val) {
                this.$parent.showCustomColumn = val
            },
            showCustomSort(val) {
                if (val) this.getCustomSortFields()
                this.$nextTick(() => (this.sortVisible = val ? true : false))
            },
            sortVisible(val) {
                this.$parent.showCustomSort = val
            },
        },
        mounted() {
            if (!this.isChangeUI) this.columnDrop()
        },
        data() {
            return {
                customColumnVisible: false,
                originalColumnList: [],
                tableHeaderList: [],
                sortVisible: false,
                excelList: [],
            }
        },
        methods: {
            // 解决多表头切换时，表头、表身错乱的问题
            generateUUID() {
                return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, function (c) {
                    let r = (Math.random() * 16) | 0,
                        v = c === 'x' ? r : (r & 0x3) | 0x8
                    return v.toString(16)
                })
            },

            // 改变列宽 -> 保存列属性按钮出现，确认后列宽更改为修改后的列宽保存到数据库
            dragendHeader(newWidth, oldWidth, column) {
                // console.log(newWidth, oldWidth, column)
                let { property, label } = column
                this.setWidth(this.tableProps, property, label, newWidth)
                this.$refs.propsIcon.style.display = 'block'
            },
            setWidth(tableProps, property, label, width) {
                tableProps.forEach((item) => {
                    if (item.children.length) this.setWidth(item.children, property, label, width)
                    if (item.field === property && item.label === label) this.$set(item, 'width', width)
                })
            },
            openColProps() {
                let _colProps = this.getColProps(this.tableProps)
                // console.log('tableProps: ', this.tableProps, '_colProps:', _colProps)
                this.$confirm('请确认是否保存表格当前的列宽和列顺序', '系统提示', {
                    confirmButtonText: '保 存',
                    cancelButtonText: '取 消',
                    type: 'warning',
                })
                    .then(() => {
                        this.saveColProp(_colProps)
                    })
                    .catch(() => {})
            },
            async saveColProp(list) {
                let { data } = await saveColProp({ list })
                if (data.ret === 1) {
                    this.$message.success(data.msg)
                } else {
                    this.$message.error(data.msg)
                }
            },
            getColProps(tableProps) {
                let _colProps = []
                tableProps.forEach((item) =>
                    item.children.length
                        ? _colProps.push({ ...this.getCommonObj(item), children: this.getColProps(item.children) })
                        : _colProps.push({ ...this.getCommonObj(item), children: [] }),
                )
                return _colProps
            },
            getCommonObj(item) {
                let _obj = {
                    viewId: this.viewId,
                    tableId: this.tableId,
                    fieldId: item.id,
                    width: item.width ? item.width : 0,
                    sort: item.sort,
                    label: item.label,
                    prop: item.field,
                }
                return _obj
            },
            // 多级表头拖动顺序(暂未支持)
            renderHeader(h, { column }) {
                return this.isChangeUI
                    ? h('span', column.label)
                    : h('span', { class: 'allowDrag', style: 'cursor:move' }, column.label)
            },
            columnDrop() {
                const wrapperTr = document.querySelector('.el-table__header-wrapper tr')
                this.sortable = Sortable.create(wrapperTr, {
                    animation: 180,
                    delay: 0,
                    handle: '.allowDrag',
                    onEnd: (evt) => {
                        // console.log('old: ', evt.oldIndex, 'new: ', evt.newIndex)
                        this.$refs.propsIcon.style.display = 'block'
                        let oldIndex = evt.oldIndex - this.leftFixedCols,
                            newIndex = evt.newIndex - this.leftFixedCols
                        this.$emit('reset-tableProps-sort', this.tableProps, oldIndex, newIndex)
                    },
                })
            },

            async getCustomCulumns() {
                let { data } = await findCustomCulumn({ tableId: this.tableId, viewId: this.viewId })
                if (data.ret === 1) {
                    this.originalColumnList = data.data || []
                    this.$nextTick(() => (this.customColumnVisible = true))
                } else {
                    this.$message.error(data.msg)
                }
            },
            closeCustomColumn() {
                this.customColumnVisible = false
            },
            async submitCustomColumn(fieldIds, originalColumnList) {
                // console.log(fieldIds, originalColumnList)
                let { data } = await saveCustomCulumn({ tableId: this.tableId, viewId: this.viewId, fieldIds })
                if (data.ret === 1) {
                    this.closeCustomColumn()
                    let _tableProps = originalColumnList.filter((v) => fieldIds.includes(v.id))
                    // console.log('_tableProps: ', _tableProps)
                    this.$emit('get-tableProps-by-customColumn', _tableProps)
                } else {
                    this.$message.error(data.msg)
                }
            },
            getCheckedCustomColumnIds(originalColumnList) {
                let _tableProps = []
                originalColumnList.forEach((item) => {
                    if (item.children.length) {
                        item.children.forEach((cItem) => {
                            if (cItem.isCustomize) {
                                item.children = item.children.filter((cItem) => cItem.isCustomize)
                                if (_tableProps.indexOf(item) === -1) _tableProps.push(item)
                            }
                        })
                    } else {
                        if (item.isCustomize) _tableProps.push(item)
                    }
                })
                // console.log(_tableProps)
                return _tableProps
            },
            getCustomSortFields() {
                this.tableHeaderList = this.tableProps
                    .map((item) => {
                        if (item.field) {
                            let prop = item.field
                            let label = item.label
                            return { prop, label }
                        }
                    })
                    .filter((item) => item)
            },
            closeCustomSort() {
                this.sortVisible = false
            },
            submitCustomSort(sortParams) {
                this.$emit('submit-sort', sortParams)
            },
            getCustomExcels() {
                // console.log('tableProps', this.tableProps)
                let _excelHeader = []
                for (let i = 0; i < this.tableProps.length; i++) {
                    const item = this.tableProps[i]
                    if (item.field === '-') {
                        let childList = []
                        if (item.children.length) {
                            for (let j = 0; j < item.children.length; j++) {
                                const cItem = item.children[j]
                                childList.push({ [cItem.label]: cItem.field })
                            }
                        }
                        _excelHeader.push({ [item.label]: childList })
                    } else {
                        _excelHeader.push({ [item.label]: item.field })
                    }
                }
                return _excelHeader
            },
            setRowClassName({ row }) {
                let { hideSelection } = row
                return hideSelection ? 'hideSelection' : 'showBorder'
            },
            handleSelect(selection, current) {
                let selected = selection.length && selection.indexOf(current) !== -1
                this.$emit('select', { selection, current, selected })
            },
            selectChange(rows) {
                this.$emit('selection-change', rows)
            },
            changeSize(pageSize) {
                this.$emit('change-size', pageSize)
            },
            goPage(pageNo) {
                this.$emit('go-page', pageNo)
            },
        },
    }
</script>

<style lang="less" scoped>
    .umy-table {
        position: relative;
        height: 100%;
        i.el-icon-document-checked {
            display: none;
            position: absolute;
            top: 6px;
            left: 3px;
            z-index: 9;
            color: #ff0036;
            font-size: 14px;
            font-weight: bold;
        }
        /deep/.plTableBox {
            height: calc(100% - 30px);
            // overflow-y: auto;
            thead {
                tr {
                    th {
                        background-color: #e0e0e0;
                    }
                }
            }
            table {
                tr {
                    td {
                        border-top: none !important;
                        border-bottom: none !important;
                        span.specialcolor {
                            color: #409eff;
                            cursor: pointer;
                        }
                        > div.cell {
                            .el-icon-arrow-right {
                                display: none;
                            }
                            .pl-tree-cell {
                                padding-left: 0;
                            }
                        }
                    }
                    &.showBorder {
                        td {
                            border-top: 1px solid #e2e1e1 !important;
                        }
                    }
                    &.hideSelection {
                        td {
                            label.el-checkbox {
                                display: none;
                            }
                        }
                    }
                    &:first-child {
                        td {
                            border-top: none !important;
                        }
                    }
                    &:last-child {
                        td {
                            border-bottom: 1px solid #e2e1e1 !important;
                        }
                    }
                }
            }
            .el-table__body-wrapper,
            .el-table__fixed-body-wrapper {
                border-bottom: 1px solid #e2e1e1;
            }
            .el-table__body-wrapper {
                overflow-y: auto;
            }
        }
    }
</style>