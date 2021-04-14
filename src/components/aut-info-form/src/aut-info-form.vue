<template>
    <el-dialog :visible.sync="visible" title="详细" :close-on-click-modal="false" :close-on-press-escape="false">
        <el-form :model="dataForm" ref="dataForm" @keyup.enter.native="visible=false" :label-width="$i18n.locale === 'en-US' ? '120px' : '80px'">
            <el-form-item v-for="(item, key, index) in label" :key="index" :label="label[key]" prop="label">' 
                <template slot-scope="scope" >
                    <aut-tag v-if="(tag[key] != null)" :dataForm="dataForm" :dataKey="key" :items="Object.keys(tag[key])" :label="tag[key]" />
                    <el-tooltip v-else class="item" effect="dark" content="点击复制" placement="top-start">
                        <el-button @click="copyItem(dataForm[key])">{{dataForm[key]}}</el-button>
                    </el-tooltip>
                </template>
            </el-form-item>
        </el-form>
        <template slot="footer">
            <el-button @click="visible = false">{{ $t('cancel') }}</el-button>
        </template>
    </el-dialog>
</template>

<script>
    import autTag from '../../aut-tag/src/aut-tag.vue'
    import VueClipboard from 'vue-clipboard2'
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
        methods: {
            copyItem(val){
                
                // 定义一个新的复制对象
                var clip = new ZeroClipboard( document.getElementById("d_clip_button"), {
                    moviePath: "ZeroClipboard.swf"
                } );
                
                // 复制内容到剪贴板成功后的操作
                clip.on( 'complete', function(client, args) {
                    alert("复制成功，复制内容为："+ args.text);
                } );
                
                
            }
        }

    }
</script>