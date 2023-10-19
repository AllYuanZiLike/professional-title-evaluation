<template>
  <div class="mod-occupation__comment">
    <el-form :inline="true" :model="state.dataForm" @keyup.enter="state.getDataList()">
      <el-form-item>
        <el-input v-model="state.dataForm.name" placeholder="评议活动名称" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-input v-model="state.dataForm.status" placeholder="活动状态" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-input v-model="state.dataForm.position" placeholder="职称" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-input v-model="state.dataForm.sGroup" placeholder="学科组" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-input v-model="state.dataForm.recommendType" placeholder="推荐类型" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-input v-model="state.dataForm.applicationType" placeholder="申报类型" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="state.getDataList()">{{ $t("query") }}</el-button>
      </el-form-item>
      <el-form-item>
        <el-button type="info" @click="state.exportHandle()">{{ $t("export") }}</el-button>
      </el-form-item>
      <el-form-item>
        <el-button v-if="state.hasPermission('occupation:comment:save')" type="primary" @click="addOrUpdateHandle()">{{ $t("add") }}</el-button>
      </el-form-item>
      <el-form-item>
        <el-button v-if="state.hasPermission('occupation:comment:delete')" type="danger" @click="state.deleteHandle()">{{ $t("deleteBatch") }}</el-button>
      </el-form-item>
    </el-form>
    <el-table v-loading="state.dataListLoading" :data="state.dataList" border @selection-change="state.dataListSelectionChangeHandle" style="width: 100%">
      <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
      <el-table-column prop="name" label="评议活动名称" header-align="center" align="center"></el-table-column>
      <el-table-column prop="info" label="评议活动介绍" header-align="center" align="center">
        <template v-slot="scope">
          <p v-html="scope.row.info"></p>
        </template>
      </el-table-column>
<!--      <el-table-column prop="superiors" label="上级评审ids" header-align="center" align="center"></el-table-column>-->
      <el-table-column prop="indicator" label="指标数" header-align="center" align="center"></el-table-column>
      <el-table-column prop="position" label="评审职称等级" header-align="center" align="center"></el-table-column>
      <el-table-column prop="sGroup" label="学科组" header-align="center" align="center"></el-table-column>
      <el-table-column prop="recommendType" label="推荐类型" header-align="center" align="center"></el-table-column>
      <el-table-column prop="applicationType" label="申报类型" header-align="center" align="center"></el-table-column>
<!--      <el-table-column prop="competitiveNum" label="竞争性参评人数（竞争型总人数）" header-align="center" align="center"></el-table-column>-->
<!--      <el-table-column prop="competitiveIndicator" label="竞争性指标数" header-align="center" align="center"></el-table-column>-->
<!--      <el-table-column prop="participantNum" label="选取人数（参与人数）" header-align="center" align="center"></el-table-column>-->
<!--      <el-table-column prop="residualIndicator" label="剩余指标数" header-align="center" align="center"></el-table-column>-->
      <el-table-column prop="stage" label="第几阶段" header-align="center" align="center"></el-table-column>
      <el-table-column prop="link" label="第几环节" header-align="center" align="center"></el-table-column>
<!--      <el-table-column prop="indicatorType" label="指标数类型（+1/等额）" header-align="center" align="center"></el-table-column>-->
      <el-table-column prop="status" label="活动状态" header-align="center" align="center"></el-table-column>
      <el-table-column :label="$t('handle')" fixed="right" header-align="center" align="center" width="150">
        <template v-slot="scope">
          <el-button v-if="state.hasPermission('occupation:comment:update')" type="primary" link @click="addOrUpdateHandle(scope.row.id)">{{ $t("update") }}</el-button>
          <el-button v-if="state.hasPermission('occupation:comment:delete')" type="primary" link @click="state.deleteHandle(scope.row.id)">{{ $t("delete") }}</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination :current-page="state.page" :page-sizes="[10, 20, 50, 100]" :page-size="state.limit" :total="state.total" layout="total, sizes, prev, pager, next, jumper" @size-change="state.pageSizeChangeHandle" @current-change="state.pageCurrentChangeHandle"> </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update :key="addKey" ref="addOrUpdateRef" @refreshDataList="state.getDataList"></add-or-update>
  </div>
</template>

<script lang="ts" setup>
import useView from "@/hooks/useView";
import { nextTick, reactive, ref, toRefs, watch } from "vue";
import AddOrUpdate from "./comment-add-or-update.vue";
import TemplateAddOrUpdate from "@/views/devtools/template-add-or-update.vue";

const view = reactive({
  getDataListURL: "/occupation/comment/page",
  getDataListIsPage: true,
  exportURL: "/occupation/comment/export",
  deleteURL: "/occupation/comment",
  deleteIsBatch: true,
  dataForm: {
    name: "",
    status: "",
    position: "",
    sGroup: "",
    recommendType: "",
    applicationType: "",
  }
});

const state = reactive({ ...useView(view), ...toRefs(view) });


const addKey = ref(0);
const addOrUpdateRef = ref();
const addOrUpdateHandle = (id?: number) => {
  addKey.value++;
  nextTick(() => {
    addOrUpdateRef.value.init(id);
  });
};
</script>
