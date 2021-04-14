<template>
    <el-dialog :visible.sync="visible" :title="!dataForm.id ? $t('add') : $t('update')" :close-on-click-modal="false" :close-on-press-escape="false">
        <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmitHandle()" :label-width="$i18n.locale === 'en-US' ? '120px' : '80px'">
            <el-form-item label="字段标签" prop="label">
                <el-input v-model="dataForm.label" placeholder="字段标签"></el-input>
            </el-form-item>
            <el-form-item label="字段名" prop="fieldName">
                <el-input v-model="dataForm.fieldName" placeholder="字段名" :disabled="true">
                    <el-select v-if="!dataForm.id" v-model="fieldData" slot="prepend" placeholder="请选择" @change="setDataField(fieldData, dataForm)">
                        <el-option v-for="(item, index) in fieldList" :key="index" :label="item.fieldName" :value="item">
                        </el-option>
                    </el-select>
                    <template v-if="dataForm.fieldName" slot="append">{{dataForm.fieldType}}</template>
                </el-input>
            </el-form-item>
            <el-form-item v-if="dataForm.fieldType" label="字段值" prop="fieldValue">
                <el-input v-if="dataForm.fieldType === 'Long' || dataForm.fieldType === 'Integer'"  v-model="dataForm.fieldValue" placeholder="字段值" oninput="value=value.replace(/[^\d]/g,'')"></el-input>
                <el-input v-else-if="dataForm.fieldType === 'Double'"  v-model="dataForm.fieldValue" placeholder="字段值" oninput="value=value.replace(/[^0-9.]/g,'')"></el-input>
                <el-input v-else  v-model="dataForm.fieldValue" placeholder="字段值"></el-input>
            </el-form-item>
            <el-form-item label="备注" prop="remark">
                <el-input v-model="dataForm.remark" placeholder="备注"></el-input>
            </el-form-item>
            <el-form-item prop="sort" label="排序">
                <el-input-number v-model="dataForm.sort" controls-position="right" :min="0" label="排序"></el-input-number>
            </el-form-item>
        </el-form>
        <template slot="footer">
            <el-button @click="visible = false">{{ $t('cancel') }}</el-button>
            <el-button type="primary" @click="dataFormSubmitHandle()">{{ $t('confirm') }}</el-button>
        </template>
    </el-dialog>
</template>

<script>
    import debounce from 'lodash/debounce'
    export default {
        data() {
            return {
                visible: false,
                dataForm: {
                    id: '',
                    setUpTypeId: '',
                    label: '',
                    fieldName: '',
                    fieldType: '',
                    fieldValue: '',
                    remark: '',
                    saveType: 0,
                    updateType: 0,
                    sort: '',
                    state: '',
                    creator: '',
                    createDate: '',
                    updater: '',
                    updateDate: ''
                },
                notUpdateName: ['fieldName', 'fieldType', 'setUpTypeId'],
                fieldList: [{
                    fieldName: 'a',
                    type: 'aa',
                    simpleType: 'bb',
                    label: "sad",
                }],
                fieldData: {
                    fieldName: 'a',
                    type: 'aa',
                    simpleType: 'bb',
                },
                setUpTypeId: 0,
            }
        },
        computed: {
            dataRule() {
                return {
                    setUpTypeId: [
                        { required: true, message: this.$t('validate.required'), trigger: 'blur' }
                    ],
                    label: [
                        { required: true, message: this.$t('validate.required'), trigger: 'blur' }
                    ],
                    fieldName: [
                        { required: true, message: this.$t('validate.required'), trigger: 'blur' }
                    ],
                    fieldValue: [
                        { required: true, message: this.$t('validate.required'), trigger: 'blur' }
                    ],
                    remark: [
                        { required: false, message: this.$t('validate.required'), trigger: 'blur' }
                    ],
                    sort: [
                        { required: true, message: this.$t('validate.required'), trigger: 'blur' }
                    ],

                }
            },
        },
        created() {
            this.setUpTypeId = this.$route.params.setUpTypeId || '0'
        },
        methods: {
            init() {
                this.visible = true
                this.$nextTick(() => {
                    this.$refs['dataForm'].resetFields()
                    if (this.dataForm.id) {
                        this.getInfo()
                    }
                    this.getRecordField()
                })
            },
            notUpdateNameHandle(dataForm) {
                let dataInfo = JSON.parse(JSON.stringify(dataForm));
                this.notUpdateName.forEach(name => delete dataInfo[name]);
                return dataInfo;
            },
            
            setDataField(item, dataForm) {
                dataForm.fieldName = item.fieldName;
                dataForm.fieldType = item.simpleType;
                dataForm.label = item.label;
                dataForm.fieldValue = ""
                console.log(item)
                console.log(dataForm.fieldType)
                console.log(this.dataForm.fieldType)
            },
            getRecordField() {
                console.log(this.setUpTypeId)
                console.log("查询field")

                this.$http
                    .get(`/sys/record/field?setUpTypeId=${this.setUpTypeId}`)
                    .then(({ data: res }) => {
                        if (res.code !== 200) {
                            return this.$message.error(res.msg);
                        }
                        this.fieldList = res.data;
                    })
                    .catch(() => {});
            },
            // 获取信息
            getInfo() {
                this.$http.get(`/sys/setup/data/${this.dataForm.id}`).then(({ data: res }) => {
                    if (res.code !== 200) {
                        return this.$message.error(res.msg)
                    }
                    this.dataForm = {
                        ...this.dataForm,
                        ...res.data
                    }
                }).catch(() => {})
            },
            // 表单提交
            dataFormSubmitHandle: debounce(function() {
                this.$refs['dataForm'].validate((valid) => {
                    if (!valid) {
                        return false
                    }
                    this.dataForm.setUpTypeId = this.setUpTypeId;
                    delete this.dataForm.fieldType
                    this.$http[!this.dataForm.id ? 'post' : 'put']('/sys/setup/data/', !this.dataForm.id ? this.dataForm : this.notUpdateNameHandle(this.dataForm))
                        .then(({ data: res }) => {
                            if (res.code !== 200) {
                                return this.$message.error(res.msg)
                            }
                            this.$message({
                                message: this.$t('prompt.success'),
                                type: 'success',
                                duration: 500,
                                onClose: () => {
                                    this.visible = false
                                    this.$emit('refreshDataList')
                                }
                            })
                        }).catch(() => {})
                })
            }, 1000, { 'leading': true, 'trailing': false })
        },

    }
</script>