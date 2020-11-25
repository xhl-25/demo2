<template lang="pug">
.register-detail-wrapper(v-loading="loading")
    h1.title 设备详情
    .detail-list
        .detail-item
            .label 设备编号：
            .info {{ detail.id }}
        .detail-item
            .label 设备名称：
            .info {{ detail.name }}
        .detail-item
            .label 设备状态：
            .info {{ ($store.getters.equipStatusObj[detail.status] || {}).name }}
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
            el-button(v-if="detail.status !== 'STATUS6'" type="primary" @click="setVisible = true") 更改状态
            el-button(@click="goList") 返回列表

    equip-set(
        :visible.sync="setVisible"
        :equip="detail"
        @success="goList"
    )
</template>

<script>
import axios from 'axios';
import EquipSet from '@/components/EquipSet.vue';

export default {
    components: { EquipSet },
    data() {
        return {
            id: this.$route.params.id,
            loading: false,
            detail: {},
            setVisible: false
        }
    },
    created() {
        axios.get('/equipment/equip-detail?id=' + this.id).then(({data}) => {
            if (data.code === 0) {
                this.setData(data.data);
                this.loading = false;
            }
        });
    },
    methods: {
        setData(val) {
            this.detail = val;
        },
        goList() {
            this.$router.push('/main/management');
        }
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
