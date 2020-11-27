<template>
    <div class="my-select">
        <div class="select-box">
            <div class="select-title">
                <!-- @keydown.down.prevent="navigateOptions('next')"
                    @keydown.up.prevent="navigateOptions('prev')" 
                    @keydown.native="listen($event)"-->
                <el-input
                    type="textarea"
                    :placeholder="placeholder"
                    v-model="searchText"
                    @input="handleInput"
                    @focus="handleFocus"
                    @click="handleTextare"
                    :disabled="isEdit"
                    @change="handleChange"
                ></el-input>
            </div>
            <div class="select-content" ref="selectContent" v-if="showContent">
                <div class="select-option">
                    <!-- 渲染父组件传递过来的值 -->
                    <div
                        class="option-item"
                        v-for="(item, index) in optionList"
                        :key="index"
                        @click="handleOption(item)"
                    >
                        {{ item.initLabel }}
                    </div>
                </div>
                <!-- <div class="add-btn">
                    <a href="javascript:void(0);" @click="clickSubjectAdd">+ 点击添加</a>
                </div> -->
            </div>
        </div>
    </div>
</template>

<script>
export default {
    name: 'editSelect',
    props: {
        optionList: Array,
        vSelectModel: String,
        showContent: Boolean,
        editIndex: Number,
        isEdit: Boolean,
        initOptionLabelList: Array,
    },
    model: {
        prop: 'vSelectModel',
        event: 'update',
    },
    data() {
        return {
            searchText: '',
            placeholder: '',
            hoverIndex: -1,
            activeOption: false,
        }
    },
    watch: {
        vSelectModel: {
            immediate: true,
            handler(newValue) {
                // console.log('select值：' + newValue)
                this.searchText = newValue
            },
        },
    },
    methods: {
        handleTextare() {
            // console.log('click select...'  click事件无效)
            this.placeholder = '请输入科目名称/科目编码快速搜索'
            this.$emit('update:showContent', !this.showContent)
            this.$refs.selectContent.style.display === 'block'
        },
        handleInput(value) {
            let target = this.searchText.trim()
            this.$emit('getTarget', target)
            this.$emit('update', value)
        },
        handleOption(item) {
            // console.log('模糊搜索点击元素...')
            let value = `${item.initLabel}`
            this.searchText = value
            this.$emit('update', value)
            this.$emit('update:showContent', false)
            // console.log(item)
            if (item.assistAccountInfo) {
                this.$emit('showSubAccountDialog', item)
            }
            this.$emit('handle-trigger')
        },
        handleFocus() {
            // console.log('focus select...')
            this.$emit('handle-focus', this.editIndex)
        },
        handleChange(value) {
            if (this.initOptionLabelList.indexOf(value) === -1) {
                value = ''
            }
            this.$emit('update', value)
        },
        // 键盘事件
        listen(event) {
            let initLen = this.optionList.length
            if (!initLen) return
            if (event.keyCode === 13 && event.ctrlKey) {
                this.searchText += '\n' //换行
            } else if (event.keyCode === 13) {
                event.preventDefault() // 阻止浏览器默认换行操作
                return
            } else if (event.keyCode === 40) {
                this.hoverIndex < initLen ? this.hoverIndex++ : (this.hoverIndex = initLen - 1)
                // console.log('键盘向下键', this.hoverIndex)
            } else if (event.keyCode === 38) {
                this.hoverIndex > 0 ? this.hoverIndex-- : (this.hoverIndex = 0)
                // console.log('键盘向上键', this.hoverIndex)
            }
            // console.log(this.optionList[this.hoverIndex])
        },
    },
}
</script>

<style lang="less" scoped>
.my-select {
    // margin: 100px 0 0 300px;
    // width: 25%;
    width: 100%;
    .select-box {
        position: relative;
        .select-title {
            // width: 100%;
            text-align: left;
            /deep/.el-textarea {
                display: block;
                width: 100%;
                textarea {
                    height: 50px;
                    padding: 0 5px;
                    border-radius: 0;
                    border: none;
                    // outline: none;
                    resize: none;
                    &:focus {
                        border: 1px solid #0079fe;
                    }
                }
            }
        }
        .select-content {
            // display: none;
            position: absolute;
            top: 50px;
            left: 0;
            z-index: 99;
            width: 100%;
            .select-option {
                max-height: 180px;
                // height: 180px;
                border: 1px solid #ccc;
                // border-top: none;
                background-color: #fff;
                overflow-y: scroll;
                .option-item {
                    height: 25px;
                    line-height: 25px;
                    padding: 0 10px;
                    overflow: hidden;
                    &:hover {
                        cursor: pointer;
                        color: #fff;
                        background-color: #0079fe;
                        // opacity: 0.5;
                    }
                }
                .activeOption {
                    color: #fff;
                    background-color: #0079fe;
                }
            }
            .add-btn {
                width: 100%;
                height: 30px;
                line-height: 30px;
                border: 1px solid #ccc;
                border-top: none;
                background-color: #fff;
                text-align: center;
                a {
                    display: inline;
                    color: #0079fe;
                    &:hover {
                        text-decoration: underline;
                    }
                }
            }
        }
    }
}
</style>
