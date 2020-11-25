<template>
<div class="register-wrapper">
    <div class="l">
        <h1 class="title">设备登记</h1>
        <search-form
            type="preStatusList"
            @search="onSearch"
        ></search-form>
    </div>
    <div class="r">
        <div class="t">
            <el-button
                type="text"
                icon="el-icon-circle-plus"
                @click="onAdd"
            >登记新设备</el-button>

            <div class="color-list">
                <div
                    class="color-item"
                    v-for="item in $store.state.preStatusList"
                    :key="item.id"
                >
                    {{ item.name }}
                    <span
                        class="color-box"
                        :style="{ 'background-color': item.color }"
                    ></span>
                </div>
            </div>
        </div>
        <div class="c" v-loading="loading">
            <el-table
                :data="list"
                style="width: 100%"
            >
                <el-table-column
                    prop="id"
                    label="设备编号"
                ></el-table-column>
                <el-table-column
                    prop="name"
                    label="设备名称"
                ></el-table-column>
                <el-table-column
                    prop="status"
                    label="设备状态"
                >
                    <template slot-scope="scope">
                        {{ getStatusInfo(scope.row.status).name }}
                        <span
                            class="color-box"
                            :style="{ 'background-color': getStatusInfo(scope.row.status).color }"
                        ></span>
                    </template>
                </el-table-column>
                <el-table-column
                    prop="category"
                    label="设备分类"
                >
                    <template slot-scope="scope">
                        {{ ($store.getters.categoryObj[scope.row.category] || {}).name }}
                    </template>
                </el-table-column>
                <el-table-column
                    prop="number"
                    label="设备数量"
                ></el-table-column>
                <el-table-column
                    prop="price"
                    label="设备单价"
                ></el-table-column>
                <el-table-column
                    prop="time"
                    label="购置时间"
                ></el-table-column>
                <el-table-column
                    label="操作"
                    width="150"
                >
                    <template slot-scope="scope">
                        <el-button size="mini" @click="viewDetail(scope.row.id)">详情</el-button>
                        <template v-if="scope.row.status === 'STATUS1'">
                            <el-button size="mini" type="primary" @click="viewEdit(scope.row.id)">编辑</el-button>
                            <el-button size="mini" type="primary" @click="viewCheck(scope.row.id)">审核</el-button>
                        </template>
                        <el-button size="mini" type="primary" v-else @click="viewApprove(scope.row.id)">入库</el-button>
                    </template>
                </el-table-column>
            </el-table>
        </div>
        <div class="b">
            <el-pagination
                @size-change="onPageSizeChange"
                @current-change="onPageNoChange"
                :current-page="pageNo"
                :page-sizes="[10, 20, 50, 100]"
                :page-size="pageSize"
                layout="total, sizes, prev, pager, next"
                :total="total"
            ></el-pagination>
        </div>
    </div>
</div>
</template>

<script>
import axios from 'axios';
import SearchForm from '@/components/SearchForm.vue';

export default {
    components: { SearchForm },
    data() {
        return {
            total: 0,
            list: [],
            pageNo: 1,
            pageSize: 10,
            searchData: {},
            loading: true
        }
    },
    methods: {
        getStatusInfo(id) {
            return this.$store.getters.preStatusObj[id];
        },
        viewDetail(id) {
            this.$router.push('/main/register-detail/' + id);
        },
        viewEdit(id) {
            this.$router.push('/main/register-edit/' + id);
        },
        viewCheck(id) {
            this.$confirm('您确认要审核通过该条信息吗?', {
                confirmButtonText: '确定审核',
                cancelButtonText: '暂不操作',
                type: 'warning'
            }).then(() => {
                this.loading = true;
                axios.post('/equipment/pre-check', {
                    id
                }).then(() => {
                    this.$message.success('审核成功');
                    this.getList();
                }).finally(() => {
                    this.loading = false;
                });
            }).catch(() => {});
        },
        viewApprove(id) {
            this.$confirm('您确认要入库通过该条信息吗?', {
                confirmButtonText: '确定入库',
                cancelButtonText: '暂不操作',
                type: 'warning'
            }).then(() => {
                this.loading = true;
                axios.post('/equipment/pre-approve', {
                    id
                }).then(() => {
                    this.$message.success('入库成功');
                    this.getList();
                }).finally(() => {
                    this.loading = false;
                });
            }).catch(() => {});
        },
        onAdd() {
            this.$router.push('/main/register-add');
        },
        onSearch(searchData) {
            this.pageNo = 1;
            this.searchData = searchData;
            this.getList();
        },
        getList() {
            this.loading = true;
            axios.post('/equipment/pre-list', {
                pageNo: this.pageNo,
                pageSize: this.pageSize,
                ...this.searchData
            }).then(({ data }) => {
                this.list = data.data.list;
                // this.list = this.list.map(item => {
                //     return {
                //         ...item,
                //         status: this.$store.getters.preStatusObj[item.status].name
                //     }
                // })
                this.total = data.data.total;
                this.loading = false;
            });
        },
        onPageSizeChange(pageSize) {
            this.pageSize = pageSize;
            this.getList();
        },
        onPageNoChange(pageNo) {
            this.pageNo = pageNo;
            this.getList();
        }
    },
    created() {
        // 设备登记列表/POST：http://jxsjs.com/equipment/pre-list
        // 参数：pageNo当前页数1开始、pageSize每页条数、id设备编号、name设备名称、status设备状态(传状态id)、category设备分类(传分类id)
        this.getList();
    }
};
</script>

<style lang="sass">
.register-wrapper
    display: flex
    .l
        flex: 0 0 300px
    .r
        flex: 1 1 auto
        margin: 0 20px
        .t
            display: flex
            justify-content: space-between
            align-items: center
            height: 60px
            padding-right: 20px
            .el-button
                font-size: 18px
        .b
            padding: 10px 0
    .color-list
        display: flex
    .color-box
        display: inline-block
        width: 10px
        height: 10px
        margin-right: 10px
    .cell
        .el-button
            padding: 5px
            & + .el-button
                margin-left: 5px
</style>
