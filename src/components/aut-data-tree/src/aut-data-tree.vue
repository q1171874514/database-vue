<template>
    <el-dialog :visible.sync="visible" title="详细" :close-on-click-modal="false" :close-on-press-escape="false">
        <el-form :model="dataForm" ref="dataForm" @keyup.enter.native="visible=false" :label-width="$i18n.locale === 'en-US' ? '120px' : '80px'">
            <el-form-item v-for="(item, key, index) in label" :key="index" :label="label[key]" prop="label">' 
                <template slot-scope="scope" >
                    <aut-tag v-if="(tag[key] != null)" :dataForm="dataForm" :dataKey="key" :items="Object.keys(tag[key])" :label="tag[key]" />
                    <el-tooltip v-else class="item" effect="dark" :content="content[key]" placement="top-start">
                        <el-button>{{dataForm[key]}}</el-button>
                    </el-tooltip>
                </template>
            </el-form-item>
        </el-form>
        <template slot="footer">
            <el-button @click="visible = false">{{ $t('cancel') }}</el-button>
        </template>
    </el-dialog>
</template>
<style lang="scss">

</style>
<script>
    import autTag from '../../aut-tag/src/aut-tag.vue'
    export default {
    components: { autTag },
        name: 'AutInfoForm',
        data() {
            return {
                visible: true

            }
        },
        props: {
            dataForm: Object,
            label: Object,
            content: Object,
            tag: Object,
        },
        mounteds: {

        }

    }
</script>