<template>
<ul class="dictionary-list">
    <li class="title">
        <div>编号</div>
        <div>名称</div>
        <div v-if="type != 'categoryList'">颜色</div>
    </li>
    <draggable v-model="list">
        <transition-group>
            <li
                v-for="(d, index) in list"
                :key="d.id"
            >
                <div>{{ d.id }}</div>
                <div
                    @dblclick="onEdit(index, 'name')"
                >
                    <input
                        :value="d.name"
                        v-if="activeItem.index === index && activeItem.type === 'name'"
                        @blur="onSave($event.target.value)"
                    >
                    <p v-else>{{ d.name }}</p>
                </div>
                <div
                    v-if="type != 'categoryList'"
                    @dblclick="onEdit(index, 'color')"
                >
                    <input
                        :value="d.color"
                        v-if="activeItem.index === index && activeItem.type === 'color'"
                        @blur="onSave($event.target.value)"
                    >
                    <p v-else>{{ d.color }}</p>
                </div>
            </li>
        </transition-group>
    </draggable>
</ul>
</template>

<script>
import axios from 'axios';
import draggable from 'vuedraggable';
export default {
    components: { draggable },
    props: ['type'],
    data() {
        return {
            list: JSON.parse(JSON.stringify(this.$store.state[this.type])),
            // 标记你编辑的是哪个单元格
            activeItem: {
                index: -1,
                type: ''    // color/name
            }
        }
    },
    watch: {
        list() {
            this.saveOrder();
        }
    },
    methods: {
        onEdit(index, type) {
            this.activeItem.index = index;
            this.activeItem.type = type;
        },
        saveOrder() {
            const { type, list } = this;
            const ids = list.map(i => i.id);
            let url = '';
            let dt = {
                ids
            };
            if (type === 'categoryList') {
                url = '/equipment/category-order';
            } else {
                url = '/equipment/status-order';
                dt.type = type === 'preStatusList' ? 'pre' : 'equip';
            }
            axios.post(url, dt).then(() => {
                if (type === 'categoryList') {
                    this.$store.commit('setCategoryList', this.list.slice());
                } else if (type === 'preStatusList') {
                    this.$store.commit('setPreStatusList', this.list.slice());
                } else if (type === 'equipStatusList') {
                    this.$store.commit('setEquipStatusList', this.list.slice());
                }

                this.$notify.success({
                    title: '排序保存成功',
                    showClose: false
                });
            });
        },
        onSave(val) {
            const { type, list, activeItem } = this;
            const currentRow = list[activeItem.index];
            let url = '';
            let dt = {
                id: currentRow.id
            };

            if (type === 'categoryList') {
                url = '/equipment/category-edit';
                dt.name = val;
            } else {
                url = '/equipment/status-edit';

                if (type === 'preStatusList') {
                    dt.type = 'pre';
                } else {
                    dt.type = 'equip';
                }

                dt.name = activeItem.type === 'name' ? val : currentRow.name;
                dt.color = activeItem.type === 'color' ? val : currentRow.color;

                // if (activeItem.type === 'name') {
                //     dt.name = val;
                //     dt.color = currentRow.color;
                // }
                // if (activeItem.type === 'color') {
                //     dt.name = currentRow.name;
                //     dt.color = val;
                // }
            }

            // http://jxsjs.com/equipment/status-edit
            // 参数：type=pre、id状态编号、 name状态名称、color状态颜色
            // http://jxsjs.com/equipment/status-edit
            // 参数：type=equip、id状态编号、 name状态名称、color状态颜色
            // http://jxsjs.com/equipment/category-edit
            // 参数：id设备分类编号、 name设备分类名称

            axios.post(url, dt).then(() => {
                this.list[activeItem.index][activeItem.type] = val;
                this.$notify.success({
                    title: '编辑成功',
                    showClose: false
                });
                if (type === 'categoryList') {
                    this.$store.commit('setCategoryList', this.list.slice());
                } else if (type === 'preStatusList') {
                    this.$store.commit('setPreStatusList', this.list.slice());
                } else if (type === 'equipStatusList') {
                    this.$store.commit('setEquipStatusList', this.list.slice());
                }
            });

            this.activeItem = {
                index: -1,
                type: ''
            };
        }
    }
    // computed: {
    //     list: {
    //         get() {
    //             return this.$store.state[this.type];
    //         },
    //         set(arr) {
    //             console.log(arr);
    //         }
    //     }
    // }
};
</script>

<style lang="sass">
.dictionary-list
    li
        display: flex
        width: 100%
        border-bottom: 1px solid #f0f0f0
        background: #fff
        transition: all .5s
        &:hover
            background: #f0f0f0
        &.title
            opacity: .6
            &:hover
                background: #fff
        & > div
            flex: 1
            text-align: center
            line-height: 40px
</style>
