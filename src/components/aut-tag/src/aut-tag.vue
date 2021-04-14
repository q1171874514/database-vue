<template>
    <div @mouseleave="mouse=false" @mousemove="mouse=true">
        <template v-for="(item, index) in items">
            <el-tag :key="index" v-if="item == dataForm[dataKey] || (mouse && updateTag)" size="small" 
                    :type="type[index]" @click="updateTag(dataForm, dataKey, item)">
                {{ label[index] }}
            </el-tag>
        </template>
    </div>
</template>
<style lang="scss">

</style>
<script>
    export default {
        name: 'AutTag',
        data() {
            return {
                mouse: false,
                type: ['', 'success', 'info', 'danger', 'warning'],
            }
        },
        props: {
            dataForm: Object,
            dataKey: String,
            items: Array,
            label: Array,
            updateTag: Function,
        },

    }
</script>