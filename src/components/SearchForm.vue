<template lang="pug">
.search-form
    h2 设备检索
    el-form(
        label-width="70px"
        :model="searchForm"
    )
        el-form-item(label="设备编号")
            el-input(
                v-model="searchForm.id"
            )
        el-form-item(label="设备名称")
            el-input(
                v-model="searchForm.name"
            )
        el-form-item(label="设备状态")
            el-select(
                v-model="searchForm.status"
            )
                el-option(
                    v-for="item in $store.state[type]"
                    :key="item.id"
                    :label="item.name"
                    :value="item.id"
                )
        el-form-item(label="设备分类")
            el-select(
                v-model="searchForm.category"
            )
                el-option(
                    v-for="item in $store.state.categoryList"
                    :key="item.id"
                    :label="item.name"
                    :value="item.id"
                )
        el-form-item
            el-button(
                type="primary"
                @click="search"
            ) 查询
</template>

<script>
export default {
    props: ['type'],
    data() {
        return {
            searchForm: {
                id: '',
                name: '',
                status: '',
                category: ''
            }
        }
    },
    methods: {
        search() {
            this.$emit('search', {
                ...this.searchForm
            });
        }
    }
};
</script>

<style lang="sass">
.search-form
    background-color: #f9f9f9
    border-radius: 5px
    padding: 15px
    .el-select, .el-button
        width: 100%
    .el-form-item:last-child
        .el-form-item__content
            margin-left: 0!important
    h2
        position: relative
        height: 50px
        padding-left: 15px
        font-size: 18px
        &:before
            content: ""
            position: absolute
            left: 0
            top: 5px
            width: 5px
            height: 16px
            background-color: #1989FA
</style>
