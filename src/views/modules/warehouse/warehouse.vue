<template>
    <el-card shadow="never" class="aui-card--fill">
        <div class="mod-warehouse__warehouse}">
            <el-form
                :inline="true"
                :model="dataForm"
                @keyup.enter.native="getDataList()"
            >
                <el-form-item>
                    <el-input
                        v-model="dataForm.id"
                        placeholder="id"
                        clearable
                    ></el-input>
                </el-form-item>
                <el-form-item>
                    <el-input
                        v-model="dataForm.name"
                        placeholder="仓库名"
                        clearable
                    ></el-input>
                </el-form-item>
                <el-form-item>
                    <el-input
                        v-model="dataForm['username']"
                        placeholder="用户名"
                        clearable
                    ></el-input>
                </el-form-item>
                <el-form-item>
                    <el-input
                        v-model="dataForm.state"
                        placeholder="状态"
                        clearable
                    >
                    <el-select  v-model="dataForm.state" slot="prepend" placeholder="请选择">
                        <el-option v-for="(item, index) in $t('warehouse.state')" :key="index" :label="item" :value="index"></el-option>
                    </el-select>
                    </el-input>
                </el-form-item>
                <el-form-item>
                    <el-button @click="getDataList()">{{
                        $t("query")
                    }}</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="info" @click="exportHandle()">{{
                        $t("export")
                    }}</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button
                        v-if="$hasPermission('warehouse:warehouse:save')"
                        type="primary"
                        @click="addOrUpdateHandle()"
                        >{{ $t("add") }}</el-button
                    >
                </el-form-item>
                <el-form-item>
                    <el-button
                        v-if="$hasPermission('warehouse:warehouse:delete')"
                        type="danger"
                        @click="deleteHandle()"
                        >{{ $t("deleteBatch") }}</el-button
                    >
                </el-form-item>
            </el-form>
            <el-table
                v-loading="dataListLoading"
                :data="dataList"
                border
                @selection-change="dataListSelectionChangeHandle"
                style="width: 100%"
            >
                <el-table-column
                    type="selection"
                    header-align="center"
                    align="center"
                    width="50"
                ></el-table-column>
                <el-table-column
                    prop="id"
                    label="id"
                    header-align="center"
                    align="center"
                ></el-table-column>
                <el-table-column
                    prop="name"
                    label="名称"
                    header-align="center"
                    align="center"
                ></el-table-column>
                <!-- <el-table-column prop="size" label="容量大小{B)" header-align="center" align="center"></el-table-column> -->
                <el-table-column
                    prop="state"
                    label="状态"
                    header-align="center"
                    align="center"
                >
                    <template slot-scope="scope">
                        <aut-tag
                            :dataForm="scope.row"
                            dataKey="state"
                            :items="[0, 1]"
                            :label="$t('warehouse.state')"
                        />
                    </template>
                </el-table-column>
                <el-table-column
                    prop="createDate"
                    label="创建时间"
                    header-align="center"
                    align="center"
                ></el-table-column>
                <el-table-column
                    prop="updateDate"
                    label="更新时间"
                    header-align="center"
                    align="center"
                ></el-table-column>
                <el-table-column
                    :label="$t('handle')"
                    fixed="right"
                    header-align="center"
                    align="center"
                    width="150"
                >
                    <template slot-scope="scope">
                        <el-button
                            v-if="$hasPermission('warehouse:warehouse:info')"
                            type="text"
                            size="small"
                            @click="infoHandle(scope.row.id)"
                            >{{ $t("info") }}</el-button
                        >
                        <el-button
                            v-if="$hasPermission('warehouse:warehouse:update')"
                            type="text"
                            size="small"
                            @click="addOrUpdateHandle(scope.row.id)"
                            >{{ $t("update") }}</el-button
                        >
                        <el-button
                            v-if="$hasPermission('warehouse:warehouse:delete')"
                            type="text"
                            size="small"
                            @click="deleteHandle(scope.row.id)"
                            >{{ $t("delete") }}</el-button
                        >
                    </template>
                </el-table-column>
            </el-table>
            <el-pagination
                :current-page="page"
                :page-sizes="[10, 20, 50, 100]"
                :page-size="limit"
                :total="total"
                layout="total, sizes, prev, pager, next, jumper"
                @size-change="pageSizeChangeHandle"
                @current-change="pageCurrentChangeHandle"
            >
            </el-pagination>
            <!-- 弹窗, 新增 / 修改 -->
            <add-or-update
                v-if="addOrUpdateVisible"
                ref="addOrUpdate"
                @refreshDataList="getDataList"
            ></add-or-update>
            <aut-info-form
                v-if="infoVisible"
                ref="autInfoForm"
                @refreshDataList="getDataList"
                :dataForm="dataInfo"
                :label="dataLabel"
                :content="dataLabel"
                :tag="$t('warehouse')"
            ></aut-info-form>
        </div>
    </el-card>
</template>

<script>
import mixinViewModule from "@/mixins/view-module";
import AddOrUpdate from "./warehouse-add-or-update";
import AutTag from "../../../components/aut-tag/src/aut-tag.vue";
import AutInfoForm from "../../../components/aut-info-form/src/aut-info-form.vue";
export default {
    mixins: [mixinViewModule],
    data() {
        return {
            mixinViewModuleOptions: {
                getDataListURL: "/warehouse/warehouse/page",
                getDataListIsPage: true,
                exportURL: "/warehouse/warehouse/export",
                deleteURL: "/warehouse/warehouse",
                deleteIsBatch: true,
            },
            dataForm: {
                id: "",
            },
            infoVisible: false,
            dataInfo: {
                id: ""
            },
            dataLabel: {
                id: "id",
                name: "仓库名称",
                size: "大小",
                state: "状态",
                userNameList: "共享用户",
                creatorUsername: "创建人",
                createDate: "创建时间",
                updaterUsername: "更新人",
                updateDate: "更新时间",
            },
            username: "",
        };
    },
    components: {
        AddOrUpdate,
        AutTag,
        AutInfoForm,
    },
    methods: {
        //详细
        infoHandle(id) {
            this.infoVisible = true;
            this.dataInfo.id = id;
            this.getInfo();
            this.$refs.autInfoForm.dataForm = this.dataInfo;
            this.$refs.autInfoForm.visible = true;
        },
        getInfo() {
            this.$http
                .get(`/warehouse/warehouse/${this.dataInfo.id}`)
                .then(({ data: res }) => {
                    if (res.code !== 200) {
                        return this.$message.error(res.msg);
                    }
                    this.dataInfo = {
                        ...this.dataInfo,
                        ...res.data,
                    };
                })
                .catch(() => {});
        },
    },
};
</script>
