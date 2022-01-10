<template>
    <div class="text-right" v-if="totalCount">
        <el-pagination
            v-if="pageSizes.length > 0"
            background
            layout="total, sizes, prev, pager, next, slot"
            :page-sizes="pageSizes"
            :total="totalCount"
            :page-size="pageSize"
            :current-page="pageNumber"
            @size-change="changeSize"
            @current-change="goPage"
        >
            <span class="pagination-slot">第 {{ pageNumber }} / {{ totalPage }} 页</span>
        </el-pagination>
        <el-pagination
            v-else
            background
            layout="total, prev, pager, next, slot"
            :total="totalCount"
            :page-size="pageSize"
            :current-page="pageNumber"
            @current-change="goPage"
        >
            <span class="pagination-slot">第 {{ pageNumber }} / {{ totalPage }} 页</span>
        </el-pagination>
    </div>
</template>
<script>
    export default {
        props: {
            pageSize: {
                type: Number,
                default: 10,
            },
            pageNumber: {
                type: Number,
                default: 1,
            },
            pageSizes: {
                type: Array,
                default: function () {
                    return [10, 30, 50, 100, 200, 500]
                },
            },
            totalPage: {
                type: Number,
                default: 1,
            },
            totalCount: {
                type: Number,
                default: 10,
            },
        },
        methods: {
            changeSize(pageSize) {
                this.$emit('change-size', pageSize)
            },
            goPage(page) {
                this.$emit('go-page', page)
            },
        },
    }
</script>
<style lang="less" scoped>
    /deep/ .el-pagination {
        height: 35px;
    }
</style>
