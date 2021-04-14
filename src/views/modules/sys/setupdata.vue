<template>
    <el-card shadow="never" class="aui-card--fill">
        <div class="mod-sys__syssetupfield}">
            <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
                <el-form-item>
                    <el-input v-model="dataForm.id" placeholder="id" clearable></el-input>
                </el-form-item>
                <el-form-item>
                    <el-button @click="getDataList()">{{ $t('query') }}</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="info" @click="exportHandle()">{{ $t('export') }}</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button v-if="$hasPermission('sys:setuptype:save')" type="primary" @click="addOrUpdateHandle()">{{ $t('add') }}</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button v-if="$hasPermission('sys:setuptype:delete')" type="danger" @click="deleteHandle()">{{ $t('deleteBatch') }}</el-button>
                </el-form-item>
            </el-form>
            <el-table v-loading="dataListLoading" :data="dataList" border @selection-change="dataListSelectionChangeHandle" style="width: 100%;">
                <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
                <el-table-column prop="label" label="字段标签" header-align="center" align="center"></el-table-column>
                <el-table-column prop="fieldName" label="字段名" header-align="center" align="center"></el-table-column>
                <el-table-column prop="fieldValue" label="字段值" header-align="center" align="center"></el-table-column>
                <el-table-column prop="fieldType" label="字段类型" header-align="center" align="center"></el-table-column>
                <!-- <el-table-column prop="remark" label="备注" header-align="center" align="center"></el-table-column> -->
                <el-table-column prop="sort" label="排序" sortable="custom" header-align="center" align="center"></el-table-column>
                <el-table-column prop="saveType" label="保存" header-align="center" align="center">
                    <template slot-scope="scope">
                        <aut-tag :dataForm="scope.row" dataKey="saveType" :items="[0, 1]" :label="$t('setup.saveType')" :updateTag="updateTag" />
                    </template>
                </el-table-column>
                <el-table-column prop="updateType" label="修改" header-align="center" align="center">
                    <template slot-scope="scope">
                        <aut-tag :dataForm="scope.row" dataKey="updateType" :items="[0, 1]" :label="$t('setup.updateType')" :updateTag="updateTag" />
                    </template>
                </el-table-column>
                <el-table-column prop="state" label="状态" header-align="center" align="center">
                    <template slot-scope="scope">
                        <aut-tag :dataForm="scope.row" dataKey="state" :items="[0, 1]" :label="$t('setup.state')" :updateTag="updateTag" />
                    </template>
                </el-table-column>
                <!-- <el-table-column prop="creator" label="创建者" header-align="center" align="center"></el-table-column> -->
                <el-table-column prop="createDate" label="创建时间" sortable="custom" header-align="center" align="center"></el-table-column>
                <!-- <el-table-column prop="updater" label="更新者" header-align="center" align="center"></el-table-column> -->
                <el-table-column prop="updateDate" label="更新时间" sortable="custom" header-align="center" align="center"></el-table-column>
                <el-table-column :label="$t('handle')" fixed="right" header-align="center" align="center" width="150">
                    <template slot-scope="scope">
                        <el-button v-if="$hasPermission('sys:setuptype:info')" type="text" size="small" @click="infoHandle(scope.row.id)">{{ $t('info') }}</el-button>
                        <el-button v-if="$hasPermission('sys:setuptype:update')" type="text" size="small" @click="addOrUpdateHandle(scope.row.id)">{{ $t('update') }}</el-button>
                        <el-button v-if="$hasPermission('sys:setuptype:delete')" type="text" size="small" @click="deleteHandle(scope.row.id)">{{ $t('delete') }}</el-button>
                    </template>
                </el-table-column>
            </el-table>
            <el-pagination :current-page="page" :page-sizes="[10, 20, 50, 100]" :page-size="limit" :total="total" layout="total, sizes, prev, pager, next, jumper" @size-change="pageSizeChangeHandle" @current-change="pageCurrentChangeHandle">
            </el-pagination>
            <!-- 弹窗, 新增 / 修改 -->
            <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList" :setUpTypeId="dataForm.setUpTypeId"></add-or-update>
            <aut-info-form v-if="infoVisible" ref="autInfoForm" @refreshDataList="getDataList" :dataForm="dataForm" :label="dataLabel" :content="dataLabel" :tag="$t('setup')"></aut-info-form>
        </div>
    </el-card>
</template>

<script>
    import mixinViewModule from '@/mixins/view-module'
    import AddOrUpdate from './setupdata-add-or-update'
    import AutTag from '../../../components/aut-tag/src/aut-tag.vue'
    import AutInfoForm from '../../../components/aut-info-form/src/aut-info-form.vue'
    export default {
        mixins: [mixinViewModule],
        data() {
            return {
                urlPath: '/sys/setup/data/',
                mixinViewModuleOptions: {
                    getDataListURL: '/sys/setup/data/page',
                    getDataListIsPage: true,
                    exportURL: '/sys/setup/data/export',
                    deleteURL: '/sys/setup/data',
                    deleteIsBatch: true
                },
                infoVisible: false,
                dataForm: {
                    id: '',
                    setUpTypeId: '0'
                },
                mouse: {
                    saveType: false,
                    updateType: false,
                    state: false,
                },
                dataLabel: {
                    id: 'Id',
                    setUpTypeId: '类id',
                    label: '标签',
                    fieldName: '字段名',
                    fieldType: '字段类型',
                    fieldValue: '字段值',
                    remark: '备注',
                    saveType: '保存类型',
                    updateType: '更新类型',
                    sort: '排序',
                    state: '状态',
                    creator: '创建人id',
                    createDate: '创建时间',
                    updater: '更新人id',
                    updateDate: '更新时间'
                }
            }
        },
        components: {
            AddOrUpdate,
            AutTag,
                AutInfoForm
        },
        created() {
            this.dataForm.setUpTypeId = this.$route.params.setUpTypeId || '0'
            this.getDataList()
        },
        methods: {
            // 新增 / 修改
            addOrUpdateHandle(id) {
                this.addOrUpdateVisible = true
                console.log(this.dataForm)
                this.$nextTick(() => {
                    this.$refs.addOrUpdate.dataForm.id = id
                    this.$refs.addOrUpdate.dataForm.setUpTypeId = this.dataForm.setUpTypeId
                    this.$refs.addOrUpdate.init()
                })
            },
            //详细
            infoHandle(id) {
                this.infoVisible = true;
                this.dataForm.id = id
                this.getInfo(id);
                this.$refs.autInfoForm.dataForm = this.dataForm
                this.$refs.autInfoForm.visible = true
            },
            getInfo(){
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
            updateTag(dataInfo, key, val) {
                let dataForm = {
                    id: dataInfo.id,
                }
                // dataForm[key] = (dataInfo[key] + 1) % numMax
                dataForm[key] = val
                this.$http['put']('/sys/setup/data/', dataForm)
                    .then(({ data: res }) => {
                        if (res.code !== 200) {
                            return this.$message.error(res.msg)
                        }
                        dataInfo[key] = dataForm[key];
                    }).catch(() => {})

            },

        }
    }
</script>