<template>
    <el-dialog :visible.sync="visible" :title="!dataForm.id ? $t('add') : $t('update')" :close-on-click-modal="false" :close-on-press-escape="false">
        <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmitHandle()" :label-width="$i18n.locale === 'en-US' ? '120px' : '80px'">
            <el-form-item label="标签" prop="label">
                <el-input v-model="dataForm.label" placeholder="标签"></el-input>
            </el-form-item>
            <el-form-item label="名称" prop="typeName">
                <el-input v-model="dataForm.typeName" placeholder="类名" :disabled="true">
                    <el-select  v-if="!dataForm.id" v-model="dataForm.typeName" slot="prepend" placeholder="请选择">
                        <el-option v-for="(item, index) in typeNameList" :key="index" :label="item" :value="item"></el-option>
                    </el-select>
                </el-input>
            </el-form-item>
            <el-form-item label="备注" prop="remark">
                <el-input v-model="dataForm.remark" placeholder="备注"></el-input>
            </el-form-item>
            <el-form-item prop="sort" label="排序">
                <el-input-number v-model="dataForm.sort" controls-position="right" :min="0" label="排序"></el-input-number>
            </el-form-item>
        </el-form>
        <template slot="footer">
            <el-button @click="visible = false">{{ $t("cancel") }}</el-button>
            <el-button type="primary" @click="dataFormSubmitHandle()">{{
                $t("confirm")
            }}</el-button>
        </template>
    </el-dialog>
</template>

<script>
    import debounce from "lodash/debounce";
    export default {
        data() {
            return {
                visible: false,
                dataForm: {
                    id: "",
                    label: "",
                    typeName: "",
                    remark: "",
                    sort: "",
                },
                typeNameList: ['a', 'b', 'c', 'd'],
                notUpdateName: ['typeName'],
            };
        },
        computed: {
            dataRule() {
                return {
                    label: [{
                        required: true,
                        message: this.$t("validate.required"),
                        trigger: "blur",
                    }, ],
                    typeName: [{
                        required: true,
                        message: this.$t("validate.required"),
                        trigger: "blur",
                    }, ],
                    remark: [{
                        required: false,
                        message: this.$t("validate.required"),
                        trigger: "blur",
                    }, ],
                    sort: [{
                        required: true,
                        message: this.$t("validate.required"),
                        trigger: "blur",
                    }, ]
                };
            },
        },
        methods: {
            init() {
                this.visible = true;
                this.$nextTick(() => {
                    this.$refs["dataForm"].resetFields();
                    if (this.dataForm.id) {
                        this.getInfo();
                    }
                    this.getRecordTypeName();
                });
            },
            notUpdateNameHandle(dataForm) {
                let dataInfo = JSON.parse(JSON.stringify(dataForm));
                this.notUpdateName.forEach(name => delete dataInfo[name]);
                return dataInfo;
            },
            getRecordTypeName() {
                console.log("查询typeName")
                this.$http
                    .get('/sys/record/typeName')
                    .then(({ data: res }) => {
                        if (res.code !== 200) {
                            return this.$message.error(res.msg);
                        }
                        this.typeNameList = res.data;
                    })
                    .catch(() => {});
            },
            // 获取信息
            getInfo() {
                this.$http
                    .get(`/sys/setup/type/${this.dataForm.id}`)
                    .then(({ data: res }) => {
                        if (res.code !== 200) {
                            return this.$message.error(res.msg);
                        }
                        this.dataForm = {
                            ...this.dataForm,
                            ...res.data,
                        };
                    })
                    .catch(() => {});
            },
            // 表单提交
            dataFormSubmitHandle: debounce(
                function() {
                    this.$refs["dataForm"].validate((valid) => {
                        if (!valid) {
                            return false;
                        }
                        this.$http[!this.dataForm.id ? "post" : "put"](
                                "/sys/setup/type/",
                                !this.dataForm.id ? this.dataForm : this.notUpdateNameHandle(this.dataForm)
                            )
                            .then(({ data: res }) => {
                                if (res.code !== 200) {
                                    return this.$message.error(res.msg);
                                }
                                this.$message({
                                    message: this.$t("prompt.success"),
                                    type: "success",
                                    duration: 500,
                                    onClose: () => {
                                        this.visible = false;
                                        this.$emit("refreshDataList");
                                    },
                                });
                            })
                            .catch(() => {});
                    });
                },
                1000, { leading: true, trailing: false }
            ),
        },
    };
</script>