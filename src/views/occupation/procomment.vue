<template>
  <div class="mod-occupation__procomment">
    <el-form :inline="true" :model="state.dataForm" @keyup.enter="state.getDataList()">
      <el-form-item>
        <el-input v-model="state.dataForm.name" placeholder="活动名称" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-date-picker v-model="state.dataForm.year" type="year" value-format="YYYY" placeholder="活动年份"></el-date-picker>
      </el-form-item>
      <el-form-item>
        <el-button @click="state.getDataList()">{{ $t("query") }}</el-button>
      </el-form-item>
      <el-form-item>
        <el-button type="info" @click="state.exportHandle()">{{ $t("export") }}</el-button>
      </el-form-item>
      <el-form-item>
        <el-button v-if="state.hasPermission('occupation:procomment:save')" type="primary" @click="addOrUpdateHandle()">{{ $t("add") }}</el-button>
      </el-form-item>
      <el-form-item>
        <el-button v-if="state.hasPermission('occupation:procomment:delete')" type="danger" @click="state.deleteHandle()">{{ $t("deleteBatch") }}</el-button>
      </el-form-item>
    </el-form>
    <el-table v-loading="state.dataListLoading" :data="state.dataList" border @selection-change="state.dataListSelectionChangeHandle" style="width: 100%">
      <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
      <el-table-column prop="name" label="活动名称" header-align="center" align="center"></el-table-column>
      <el-table-column prop="info" label="活动简介" header-align="center" align="center"></el-table-column>
      <el-table-column prop="yearDate" label="活动年份" header-align="center" align="center"></el-table-column>
      <el-table-column :label="$t('handle')" fixed="right" header-align="center" align="center" width="150">
        <template v-slot="scope">
          <el-button type="primary" link @click="selectedProList(scope.row.id)">查看已选专家列表</el-button>
          <el-button v-if="state.hasPermission('occupation:procomment:update')" type="primary" link @click="addOrUpdateHandle(scope.row.id)">{{ $t("update") }}</el-button>
          <el-button v-if="state.hasPermission('occupation:procomment:delete')" type="primary" link @click="state.deleteHandle(scope.row.id)">{{ $t("delete") }}</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination :current-page="state.page" :page-sizes="[10, 20, 50, 100]" :page-size="state.limit" :total="state.total" layout="total, sizes, prev, pager, next, jumper" @size-change="state.pageSizeChangeHandle" @current-change="state.pageCurrentChangeHandle"> </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update :key="addKey" ref="addOrUpdateRef" @refreshDataList="state.getDataList"></add-or-update>
    <proJudgeList ref="selectedProListRef" @refreshDataList="state.getDataList"></proJudgeList>
  </div>
</template>

<script lang="ts" setup>
import useView from "@/hooks/useView";
import { nextTick, reactive, ref, toRefs, watch } from "vue";
import AddOrUpdate from "./procomment-add-or-update.vue";
import proJudgeList from "@/views/occupation/proJudgeList.vue";

const view = reactive({
  getDataListURL: "/occupation/procomment/page",
  getDataListIsPage: true,
  exportURL: "/occupation/procomment/export",
  deleteURL: "/occupation/procomment",
  deleteIsBatch: true,
  daterange: null,
  dataForm: {
    name: "",
    year:"",
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

const selectedProListRef = ref();
const selectedProList = (id:number)=>{
  nextTick(()=>{
    selectedProListRef.value.init(id);
  })
}
</script>
