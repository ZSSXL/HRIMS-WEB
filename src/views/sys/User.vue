<template>
    <div>
        <el-row>
            <el-col :span="6">
                <div class="mt-4">
                    <el-input placeholder="输入名称查询" v-model="queryUserName" class="input-with-select">
                        <template #append>
                            <el-button type="primary" @click="queryByUserName">
                                <el-icon><Search /></el-icon>
                            </el-button>
                        </template>
                    </el-input>
                </div>
            </el-col>
            <el-col :span="2" :offset="1">
                <el-button v-if="sort" type="primary" @click="pageSortUp">
                    <el-icon><SortDown /></el-icon>
                </el-button>
                <el-button v-else type="primary" @click="pageSortDown">
                    <el-icon><SortUp /></el-icon>
                </el-button>
            </el-col>
            <el-col :span="2" :offset="12">
                <el-button type="primary">
                    <el-icon><CirclePlus /></el-icon>
                    添加员工
                </el-button>
                <span>{{$route.name}}</span>
            </el-col>
        </el-row>
        <el-row>
            <el-col :span="24">
                <el-table :data="filterTableData" style="width: 100%">
                    <el-table-column type="index" min-width="120" />
                    <el-table-column prop="username" label="姓名" min-width="120" />
                    <el-table-column prop="loginName" label="登录名" min-width="120" />
                    <el-table-column prop="email" label="邮箱" min-width="240" />
                    <el-table-column prop="password" label="密码" min-width="120" />
                    <el-table-column prop="createTime" label="创建时间" min-width="120" />
                    <el-table-column fixed="right" min-width="120">
                        <template #header>
                            <el-input v-model="search" size="small" placeholder="当前查询名称" />
                        </template>
                        <template #default="scope">
                            <el-button link type="primary" size="small" @click="detailUser(scope.$index)">详情</el-button>
                            <el-button link type="danger" size="small" @click="deleteUser(scope.$index)">删除</el-button>
                        </template>
                    </el-table-column>
                </el-table>
            </el-col>
        </el-row>
        <el-row style="margin-top: 1rem">
            <el-col :span="6" :offset="12">
                <el-pagination v-model:page-size="pageInfo.size" v-model:currentPage="pageInfo.current" :pager-count="7" layout="prev, pager, next" :page-count="pageInfo.pages" @current-change="currentChange" />
            </el-col>
        </el-row>
    </div>
</template>

<script>
import { computed, reactive, onBeforeMount, toRefs } from 'vue'
import { Confirm } from 'notiflix/build/notiflix-confirm-aio'
import { Notify } from 'notiflix/build/notiflix-notify-aio'
import { Search, SortUp, SortDown, CirclePlus } from '@element-plus/icons-vue'
import { userList } from '@/api/request/user.js'
import { useRouter } from 'vue-router'
import { useStore } from 'vuex'

export default {
    name: 'SysUser',
    components: {
        Search,
        SortUp,
        SortDown,
        CirclePlus
    },
    setup() {
        const router = useRouter()
        const store = useStore()

        const userState = reactive({
            pageInfo: {
                total: 4,
                size: 20,
                current: 1,
                pages: 50
            },
            search: '',
            queryUserName: '',
            sort: true,
            userList: [],
            filterTableData: []
        })

        onBeforeMount(() => {
            getUserList(userState.sort, userState.queryUserName, userState.pageInfo.current, userState.pageInfo.size)
        })

        const getUserList = async (sort, username, pageNumber, pageSize) => {
            if (!pageNumber || pageNumber < 1) {
                pageNumber = 1
            }
            if (!pageSize || pageSize < 0) {
                pageSize = 20
            }
            const param = { sort: sort, name: username, pageIndex: pageNumber, pageSize: pageSize }
            let response = await userList(param)
            if (response.status === 0) {
                buildData(response.data)
            } else {
                Notify.failure(response.msg)
            }
        }

        const buildData = (data) => {
            userState.userList = data.records
            userState.pageInfo.total = data.total
            userState.pageInfo.size = data.size
            userState.pageInfo.current = data.current
            userState.pageInfo.pages = data.pages
        }

        /**
         * 用户详情
         */
        const detailUser = (index) => {
            console.log('查看用户详情......', index)
            console.log(userState.filterTableData[index].userId)
            store.commit('setUserId', userState.filterTableData[index].userId)
            router.push({ path: '/detail' })
        }

        /**
         * 删除用户
         */
        const deleteUser = (index) => {
            Confirm.show('警告', '确定删除该用户?', '删除', '取消', () => {
                console.log('DeleteUserId: ', userState.filterTableData[index].userId)
            })
        }

        /**
         * 排序查询
         */
        const pageSortUp = () => {
            userState.sort = false
            getUserList(false, userState.queryUserName, 1, userState.pageInfo.size)
        }

        /**
         * 排序查询
         */
        const pageSortDown = () => {
            userState.sort = true
            getUserList(true, userState.queryUserName, 1, userState.pageInfo.size)
        }

        /**
         * 分页条状
         */
        const currentChange = (val) => {
            getUserList(userState.sort, userState.queryUserName, val, userState.pageInfo.size)
        }

        /**
         * 名称查询 -- 模糊匹配
         */
        const queryByUserName = () => {
            getUserList(userState.sort, userState.queryUserName, 1, userState.pageInfo.size)
        }

        userState.filterTableData = computed(() => userState.userList.filter((data) => !userState.search || data.username.toLowerCase().includes(userState.search.toLowerCase())))

        return {
            ...toRefs(userState),
            deleteUser,
            detailUser,
            currentChange,
            pageSortUp,
            pageSortDown,
            queryByUserName
        }
    }
}
</script>

<style lang="scss"></style>
