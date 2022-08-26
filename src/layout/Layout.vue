<template>
    <div class="common-layout">
        <el-container>
            <el-aside class="app-main-side" :style="defaultHeight" :width="asideWidth">
                <el-affix :z-index="1200">
                    <div class="aside-logo" @click="onRefresh">
                        <el-image class="logo-image" :src="logo" fit="contain" />
                        <span v-show="!isCollapse" class="app-title">
                            <span>HRIMS</span>
                        </span>
                    </div>
                </el-affix>
                <el-menu router :default-active="$route.path" :collapse="isCollapse" :collapse-transition="false" @select="selectPage">
                    <div v-for="menu in routers" :key="menu">
                        <el-menu-item v-if="menu.children" :index="menu.children[0].path">
                            <el-icon>
                                <component :is="menu.children[0].icon"></component>
                            </el-icon>
                            <template #title>
                                {{ menu.children[0].name }}
                            </template>
                        </el-menu-item>
                    </div>
                </el-menu>
            </el-aside>
            <el-container>
                <el-header>
                    <div class="app-main-header">
                        <div class="app-side-btn">
                            <div @click="onCollapse" v-if="isCollapse">
                                <el-icon>
                                    <expand />
                                </el-icon>
                            </div>
                            <div @click="onCollapse" v-else>
                                <el-icon>
                                    <fold />
                                </el-icon>
                            </div>
                        </div>
                        <div>
                            <span style="font-weight: 400; color: #606266; cursor: text">{{ $route.name }}</span>
                        </div>
                        <div class="app-quit" @click="signout">退出</div>
                    </div>
                </el-header>
                <el-main>
                    <router-view v-slot="{ Component }">
                        <keep-alive>
                            <component :is="Component" v-if="$route.meta.keepAlive" :key="$route.name" />
                        </keep-alive>
                        <component :is="Component" v-if="!$route.meta.keepAlive" :key="$route.name" />
                        <!-- <component :is="Component" :key="$route.name" /> -->
                    </router-view>
                </el-main>
            </el-container>
        </el-container>
    </div>
</template>

<script>
import { reactive, toRefs, onBeforeMount } from 'vue'
import { useRouter } from 'vue-router'
import { User, OfficeBuilding, Setting, Location, Tickets, Expand, Fold } from '@element-plus/icons-vue'

export default {
    name: 'Layout',
    components: {
        User,
        OfficeBuilding,
        Location,
        Tickets,
        Setting,
        Expand,
        Fold
    },
    setup() {
        const router = useRouter()
        // 侧边菜单栏是否展开
        const state = reactive({
            logo: require('@/assets/images/logo.png'),
            isCollapse: false,
            asideWidth: '160px',
            defaultHeight: {
                height: ''
            },
            routers: []
        })

        onBeforeMount(() => {
            let token = localStorage.getItem('hrims-token')
            if (token == null || token.length == 0) {
                // router.push({ path: '/login' })
                console.log('请完成登录')
            }
            const myRouters = router.options.routes
            const currentRole = localStorage.getItem('hrims-role')
            myRouters.forEach((element) => {
                if (element.meta) {
                    if ('1' === currentRole && element.meta.requireAdmin) {
                        state.routers.push(element)
                    } else if ('0' === currentRole && !element.meta.requireAdmin) {
                        state.routers.push(element)
                    }
                }
            })

            const currentPage = sessionStorage.getItem('hrims-current-page')
            let paths = state.routers.map((item) => {
                return item.path
            })
            let currentRoutePath = state.routers[0].path
            if (currentPage) {
                if (paths.includes(currentPage)) {
                    router.push({ path: currentPage })
                } else {
                    router.push({ path: currentRoutePath })
                }
            } else {
                router.push({ path: currentRoutePath })
            }
            state.defaultHeight.height = document.body.clientHeight + 'px'
        })

        const onRefresh = () => {
            console.log('回到主页面')
            // router.push({ path: 'main' })
        }

        const selectPage = (index) => {
            sessionStorage.setItem('hrims-current-page', index)
            router.push({ path: index })
        }

        const signout = () => {
            localStorage.removeItem('hrims-token')
            router.push({ path: '/login' })
        }

        const onCollapse = () => {
            if (state.isCollapse) {
                state.asideWidth = '160px'
                state.isCollapse = false
                console.log(state.defaultHeight.height)
            } else {
                state.asideWidth = '64px'
                state.isCollapse = true
                console.log(state.defaultHeight.height)
            }
        }

        return { ...toRefs(state), onCollapse, onRefresh, selectPage, signout }
    }
}
</script>

<style>
.el-header {
    --el-header-padding: 2px !important;
}

.el-menu {
    border-right: none !important;
}

.aside-logo {
    height: 3.5rem;
    cursor: pointer;
    /* border-bottom: 1px #cccccc solid; */
}

.logo-image {
    width: 40px;
    height: 40px;
    top: 12px;
    display: flex;
    padding-left: 12px;
    align-items: center;
}

.app-main-header {
    background: #ffffff;
    color: #333333;
    display: flex;
    align-items: center;
    padding: 0 1.5rem;
    height: 3.5rem;
    width: 100%;
    position: relative;
    box-shadow: 0 0 13px 0 rgb(0 0 0 / 7%);
    z-index: 999;
}

.app-main-side {
    background: #ffffff;
    color: #333333;
    box-shadow: 4px 0 5px rgb(0 0 0 / 7%);
}

.app-side-btn {
    padding-right: 0.83rem;
    padding-top: 0.45rem;
    font-size: 1.25rem;
    color: #004450;
    font-weight: 700;
    display: flex;
    align-items: center;
    top: 1rem;
}

.app-title {
    padding-left: 0.83rem;
    font-size: 1.25rem;
    color: #004450;
    font-weight: 700;
}

.app-quit {
    cursor: pointer;
    padding-right: 0.83rem;
    font-size: 1.25rem;
    color: #004450;
    font-weight: 700;
    margin-right: 1rem;
    margin-left: auto;
    align-items: center;
    justify-content: flex-end;
}
</style>
