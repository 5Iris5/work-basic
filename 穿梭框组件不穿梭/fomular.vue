<template>
    <div id="asset-echart">
        <div class="echart-contr">
            <el-button type="primary" @click="createOne" size="mini">快速建表</el-button>
        </div>
        <div id="spread" ref="spread"></div>
        <dialogExcel
            :excelVisible="excelVisible"
            :dialogTitle="dialogTitle"
            @cancel-excel-method="cancelExcelMethod"
            @confirm-excel-method="cofirmExcelMethod"
        ></dialogExcel>
    </div>
</template>

<script>
    import dialogExcel from '@/components/dialog/dialogExcel'

    import jexcelStyle from 'jexcel/dist/jexcel.css' // eslint-disable-line no-unused-vars
    import jexcel from 'jexcel' // eslint-disable-line no-unused-vars
    import jSuites from 'jSuites'
    let mergeStart = null
    let toolbar = [
        {
            type: 'i',
            content: 'undo',
            onclick: function (element, instance) {
                instance.undo()
            },
        },
        {
            type: 'i',
            content: 'redo',
            onclick: function (element, instance) {
                instance.redo()
            },
        },
        {
            type: 'i',
            content: 'cloud_download',
            onclick: function (element, instance) {
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
            onclick: function (element, instance) {
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
            onclick: function (element, instance, item) {
                if (!item.color) {
                    // console.log(jSuites.color)
                    let colorPicker = jSuites.color(item, {
                        onchange: function (o, v) {
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
            onclick: function (element, instance, item) {
                if (!item.color) {
                    let colorPicker = jSuites.color(item, {
                        onchange: function (o, v) {
                            cellPicker(o, v, instance, 'backgroundColor')
                        },
                    })
                    colorPicker.open()
                }
            },
        },
    ]
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
    let cellPicker = function (o, v, instance, style) {
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

    export default {
        components: { dialogExcel },
        data() {
            return {
                excelVisible: false,
                dialogTitle: '快速建表',
                minDimensions: [26, 34], // 表格初始行列总数[y, x]
            }
        },
        mounted() {
            this.createJExcel()
        },
        methods: {
            // 快速建表
            createOne() {
                this.excelVisible = true
            },
            cancelExcelMethod() {
                this.excelVisible = false
            },
            cofirmExcelMethod(data) {
                console.log(data)
                this.excelVisible = false
            },

            createJExcel() {
                let options = {
                    data: [[]],
                    minDimensions: this.minDimensions,
                    allowToolbar: true,
                    toolbar: toolbar,
                    text: text,
                }
                let spreadsheet = jexcel(this.$refs['spread'], options)
                Object.assign(this, spreadsheet)
            },
        },
    }
</script>

<style lang="less" scoped>
    @import '../../../assets/font/jexcelFonts.css';
    #asset-echart {
        padding: 10px;
        .echart-contr {
            margin-bottom: 10px;
        }
        /deep/.jcolor table {
            width: auto;
        }
    }
</style>
