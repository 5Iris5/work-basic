<!--
 * @Author: your name
 * @Date: 2021-12-16 10:01:57
 * @LastEditTime: 2022-01-05 11:45:07
 * @LastEditors: Please set LastEditors
 * @Description: 打开koroFileHeader查看配置 进行设置: https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE
 * @FilePath: \oa_web\src\components\view\customTable.vue
-->
<template>
    <div class="custom-table">
        <el-table
            ref="table"
            highlight-current-row
            header-cell-class-name="tableHeader"
            size="mini"
            border
            :data="tableDataList"
            height="calc(100% - 30px)"
            @selection-change="selectChange"
        >
            >
            <el-table-column type="selection"> </el-table-column>
            <el-table-column
                v-for="(item, idx) in tableProps"
                :key="idx"
                show-overflow-tooltip
                :prop="item.field"
                :label="item.label"
                :align="textAlignRightList.includes(item.field) ? 'right' : 'left'"
                :fixed="colFixedList.includes(item.field) ? 'left' : false"
            >
                <template slot-scope="scope">
                    <slot
                        name="normal"
                        :row="{ tit: item, data: scope.row, prop: item.field, $index: scope.$index }"
                    ></slot>
                </template>
            </el-table-column>
            <el-table-column v-if="showOperate" :label="operateLabel" :width="operateWidth">
                <template slot-scope="scope"><slot name="oprate" :row="scope.row"></slot></template>
            </el-table-column>
        </el-table>
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
    import { findCustomCulumn, saveCustomCulumn } from '@/api/system/menu'
    export default {
        name: 'customTable',
        components: { customColumn, Pagination, customSort },
        props: {
            columnList: {
                type: Array,
                default: () => [],
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
            textAlignRightList: {
                type: Array,
                default: () => [],
            },
            colFixedList: {
                type: Array,
                default: () => [],
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
            tableDataList() {
                return this.tableData.list || this.tableData
            },
            tableProps() {
                return this.columnList.filter((v) => v.isCustomize)
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
        data() {
            return {
                customColumnVisible: false,
                originalColumnList: [],
                tableHeaderList: [],
                sortVisible: false,
            }
        },
        methods: {
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
                // console.log('fieldIds: ', fieldIds)
                let { data } = await saveCustomCulumn({ tableId: this.tableId, viewId: this.viewId, fieldIds })
                if (data.ret === 1) {
                    this.closeCustomColumn()
                    // 重写table的表头字段(非多级表头)
                    let _tableProps = originalColumnList.filter((v) => fieldIds.includes(v.id))
                    this.$emit('get-tableProps-by-customColumn', _tableProps)
                } else {
                    this.$message.error(data.msg)
                }
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
                let _excelHeader = this.tableProps
                    .map((item) => {
                        if (item.field) return { [item.label]: item.field }
                    })
                    .filter((item) => item)
                return _excelHeader
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
    .custom-table {
        height: 100%;
        /deep/.el-table {
            height: calc(100% - 30px);
            overflow-y: auto;
        }
    }
</style>