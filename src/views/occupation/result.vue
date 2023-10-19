<template>
  <div class="mod-occupation__result">
    <el-form :inline="true" :model="state.dataForm" @keyup.enter="state.getDataList()">
      <el-form-item>
        <el-input v-model="state.dataForm.commentId" placeholder="评审活动" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-input v-model="state.dataForm.participantId" placeholder="参与人" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-input v-model="state.dataForm.result" placeholder="推荐意见（结果）" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-input v-model="state.dataForm.count" placeholder="得票数" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="state.getDataList()">{{ $t("query") }}</el-button>
      </el-form-item>
      <el-form-item>
        <el-button type="info" @click="state.exportHandle()">{{ $t("export") }}</el-button>
      </el-form-item>
      <el-form-item>
        <el-button v-if="state.hasPermission('occupation:result:save')" type="primary" @click="addOrUpdateHandle()">{{ $t("add") }}</el-button>
      </el-form-item>
      <el-form-item>
        <el-button v-if="state.hasPermission('occupation:result:delete')" type="danger" @click="state.deleteHandle()">{{ $t("deleteBatch") }}</el-button>
      </el-form-item>
    </el-form>
    <el-table v-loading="state.dataListLoading" :data="state.dataList" border @selection-change="state.dataListSelectionChangeHandle" style="width: 100%">
      <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
      <el-table-column prop="commentId" label="评审活动" header-align="center" align="center"></el-table-column>
      <el-table-column prop="participantId" label="参与人" header-align="center" align="center"></el-table-column>
      <el-table-column prop="result" label="推荐意见（结果）" header-align="center" align="center"></el-table-column>
      <el-table-column prop="count" label="得票数" header-align="center" align="center"></el-table-column>
      <el-table-column :label="$t('handle')" fixed="right" header-align="center" align="center" width="150">
        <template v-slot="scope">
          <el-button v-if="state.hasPermission('occupation:result:update')" type="primary" link @click="addOrUpdateHandle(scope.row.id)">{{ $t("update") }}</el-button>
          <el-button v-if="state.hasPermission('occupation:result:delete')" type="primary" link @click="state.deleteHandle(scope.row.id)">{{ $t("delete") }}</el-button>
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
import AddOrUpdate from "./result-add-or-update.vue";

const view = reactive({
  getDataListURL: "/occupation/result/page",
  getDataListIsPage: true,
  exportURL: "/occupation/result/export",
  deleteURL: "/occupation/result",
  deleteIsBatch: true,
  dataForm: {
    commentId: "",
    participantId: "",
    result: "",
    count: "",
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