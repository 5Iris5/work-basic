<template>
    <div id="tag-view">
        <scroll-pane ref="scrollPane" class="tags-view-wrapper" @scroll="handleScroll">
            <router-link
                v-for="tag in cachedViews"
                ref="tag"
                :key="tag.path"
                :to="{ path: tag.path, name: tag.name, fullPath: tag.fullPath }"
                tag="span"
                class="tags-view-item"
                :class="isActive(tag) ? 'active' : ''"
                @click.middle.native="closeSelectedTag(tag)"
                @contextmenu.prevent.native="openMenu(tag, $event)"
            >
                {{ tag.title }}
                <span class="el-icon-close" @click.prevent.stop="closeSelectedTag(tag)" />
            </router-link>
        </scroll-pane>
        <ul v-show="visible" class="contextmenu" :style="{ left: left + 'px', top: top + 'px' }">
            <li @click="refreshSelectedTag(selectedTag)">刷新</li>
            <li @click="closeSelectedTag(selectedTag)">关闭</li>
            <li @click="closeOthersTags">关闭其他</li>
            <li @click="closeAllTags(selectedTag)">关闭所有</li>
        </ul>
    </div>
</template>

<script>
import ScrollPane from './scrollPane'
export default {
    name: 'tagView',
    components: { ScrollPane },
    data() {
        return {
            visible: false,
            top: 0,
            left: 0,
            selectedTag: {},
        }
    },
    computed: {
        cachedViews() {
            // console.log('computed: ', this.$store.state.cachedViews)
            return this.$store.state.cachedViews
        },
    },
    watch: {
        $route() {
            // this.$store.state.cachedViews = []
            // this.$store.commit('setCachedView')
            // console.log('tagsView点击路由信息：', this.$route)
            this.addTags(this.$route)
            this.moveToCurrentTag()
        },
        visible(value) {
            if (value) {
                document.body.addEventListener('click', this.closeMenu)
            } else {
                document.body.removeEventListener('click', this.closeMenu)
            }
        },
    },
    mounted() {
        //在页面加载时读取sessionStorage里的状态信息
        if (sessionStorage.getItem('store')) {
            this.$store.state.cachedViews = JSON.parse(sessionStorage.getItem('store'))
            // console.log('mounted: ', this.$store.state.cachedViews)
        }
        // if (sessionStorage.getItem('scrollPosition')) {
        //     this.$store.state.scrollPosition = JSON.parse(sessionStorage.getItem('scrollPosition'))
        //     console.log('mounted: ', this.$store.state.cachedViews)
        // }
    },
    methods: {
        isActive(route) {
            return route.path === this.$route.path
        },
        addTags(route) {
            let obj = {
                title: route.meta.title,
                path: route.path,
                name: route.name,
                fullPath: route.fullPath,
            }
            this.$store.dispatch('addCachedView', obj)
            // console.log('添加标签路由: ', obj)
        },
        closeSelectedTag(view) {
            // console.log('路由信息：', view)
            this.closeMenu()
            this.$store.dispatch('delCachedView', view).then(({ cachedViews }) => {
                if (this.isActive(view)) {
                    this.toLastView(cachedViews, view)
                }
            })
        },
        closeOthersTags() {
            this.$router.push(this.selectedTag.path)
            this.$store.dispatch('delOthersCachedViews', this.selectedTag).then(() => {
                this.moveToCurrentTag()
            })
        },
        closeAllTags(view) {
            this.$store.dispatch('delAllViews').then(({ cachedViews }) => {
                this.toLastView(cachedViews, view)
            })
        },
        toLastView(cachedViews, view) {
            // console.log('dispatch触发得到的删除后剩余标签：', cachedViews, '当前点击标签: ', view)
            if (cachedViews.length === 0) {
                this.$router.push('/home')
                return
            }
            let latestView = cachedViews.slice(-1)[0]
            if (latestView) {
                this.$router.push(latestView.path)
            }
        },
        moveToCurrentTag() {
            let tags = this.$refs.tag
            if (!tags) return
            this.$nextTick(() => {
                for (let tag of tags) {
                    if (tag.to.path === this.$route.path) {
                        this.$refs.scrollPane.moveToTarget(tag)
                        // when query is different then update
                        if (tag.to.path !== this.$route.path) {
                            console.log(tag.to.path, this.$route.path)
                            this.$store.dispatch('updateView', this.$route)
                        }
                        break
                    }
                }
            })
        },
        refreshSelectedTag(view) {
            // 刷新之前保存路由，避免丢失
            sessionStorage.setItem('store', JSON.stringify(this.$store.state.cachedViews))
            // sessionStorage.setItem('scrollPosition', JSON.stringify(this.$store.state.scrollPosition))
            this.$store.dispatch('delCachedView', view).then(cachedViews => {
                // console.log('更新路由', cachedViews)
                this.$nextTick(() => {
                    let { path } = view
                    this.$router.replace({
                        path: '/redirect' + path,
                    })
                    this.$store.dispatch('addCachedView', view)
                })
            })
        },

        openMenu(tag, e) {
            let menuMinWidth = 100 // tags width
            let offsetLeft = this.$el.getBoundingClientRect().left // container margin-left
            let offsetWidth = this.$el.offsetWidth // container width
            let left = e.clientX - offsetLeft + 10 // margin-left
            let maxLeft = offsetWidth - menuMinWidth // margin-right
            // console.log(this.$el, offsetLeft, offsetWidth, left, maxLeft)
            if (left > maxLeft) {
                this.left = maxLeft
            } else {
                this.left = left
            }
            this.top = this.$el.getBoundingClientRect().height - 10
            this.visible = true
            this.selectedTag = tag
            // console.log('点击的导航标签: ', tag)
        },
        closeMenu() {
            this.visible = false
        },
        handleScroll() {
            this.closeMenu()
        },
    },
}
</script>

<style lang="less" scoped>
#tag-view {
    position: relative;
    top: 0;
    left: 0;
    z-index: 999;
    height: 32px;
    width: 100%;
    background: #fff;
    // border-bottom: 1px solid #d8dce5;
    box-shadow: 0 1px 3px 0 rgba(0, 0, 0, 0.12), 0 0 3px 0 rgba(0, 0, 0, 0.04);
    .tags-view-item {
        position: relative;
        display: inline-block;
        cursor: pointer;
        height: 26px;
        line-height: 26px;
        border: 1px solid #d8dce5;
        border-radius: 2px;
        color: #495060;
        background: #fff;
        padding: 0 8px;
        font-size: 12px;
        margin-left: 5px;
        margin-top: 4px;
        &:first-of-type {
            margin-left: 15px;
        }
        &:last-of-type {
            margin-right: 15px;
        }
        &.active {
            color: #fff;
            // background-color: #42b983;
            // border-color: #42b983;
            background-color: #409eff;
            border-color: #409eff;
            &::before {
                content: '';
                background: #fff;
                display: inline-block;
                width: 8px;
                height: 8px;
                border-radius: 50%;
                position: relative;
                margin-right: 2px;
            }
        }
        .el-icon-close {
            width: 16px;
            height: 16px;
            vertical-align: 2px;
            border-radius: 50%;
            text-align: center;
            transition: all 0.3s cubic-bezier(0.645, 0.045, 0.355, 1);
            transform-origin: 100% 50%;
            &:before {
                transform: scale(0.6);
                display: inline-block;
                vertical-align: -3px;
            }
            &:hover {
                background-color: #b4bccc;
                color: #fff;
            }
        }
    }
    .contextmenu {
        margin: 0;
        background: #fff;
        z-index: 3000;
        position: absolute;
        list-style-type: none;
        padding: 5px 0;
        border-radius: 4px;
        font-size: 12px;
        font-weight: 400;
        color: #333;
        box-shadow: 2px 2px 3px 0 rgba(0, 0, 0, 0.3);
        li {
            margin: 0;
            padding: 7px 16px;
            cursor: pointer;
            &:hover {
                background: #eee;
            }
        }
    }
}
</style>
