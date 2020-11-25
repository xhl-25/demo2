<template lang="pug">
.register-detail-wrapper(v-loading="loading")
    h1.title 设备登记详情
    .detail-list
        .detail-item
            .label 设备编号：
            .info {{ detail.id }}
        .detail-item
            .label 设备名称：
            .info {{ detail.name }}
        .detail-item
            .label 设备状态：
            .info {{ ($store.getters.preStatusObj[detail.status] || {}).name }}
        .detail-item
            .label 设备分类：
            .info {{ ($store.getters.categoryObj[detail.category] || {}).name }}
        .detail-item
            .label 数量：
            .info {{ detail.number }}
        .detail-item
            .label 单价：
            .info {{ detail.price }}
        .detail-item
            .label 购置日期：
            .info {{ detail.time }}
        .detail-item
            template(v-if="detail.status === 'STATUS1'")
                el-button(type="primary" @click="viewEdit") 编辑
                el-button(type="primary" @click="viewCheck") 审核
            el-button(@click="goList") 返回列表
</template>

<script>
import RegisterMixin from '@/mixins/register.js';
export default {
    mixins: [ RegisterMixin ],
    data() {
        return {
            loading: true,
            detail: {}
            // this.$route.query
        }
    },
    methods: {
        setData(val) {
            this.detail = val;
        },
        goList() {
            this.$router.push('/main/register');
        },
        viewEdit() {
            this.$router.push('/main/register-edit/' + this.detail.id);
        },
        viewCheck() {
            this.$confirm('您确认要审核通过该条信息吗?', {
                confirmButtonText: '确定审核',
                cancelButtonText: '暂不操作',
                type: 'warning'
            }).then(() => {
                axios.post('/equipment/pre-check', {
                    id: this.detail.id
                }).then(() => {
                    this.$message.success('审核成功');
                    this.goList();
                });
            }).catch(() => {});
        },
    }
};
</script>

<style lang="sass">
.register-detail-wrapper
    .detail-list
        margin-right: 20px
        padding: 10px 20px
        background: #f0f0f0
        border-radius: 4px
        .detail-item
            display: flex
            padding: 10px 0
            &:first-child
                border-bottom: 1px solid #ccc
                padding-bottom: 20px
            &:nth-child(2), &:last-child
                padding-top: 20px
            .label
                width: 100px
                padding-right: 20px
                text-align: right
                color: #999
</style>
