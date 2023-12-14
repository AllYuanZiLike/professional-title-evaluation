<template>
  <div class="mod-occupation__categorie">
    <el-drawer v-model="drawerHistory"  title="历史职评活动" direction="rtl" size="85%">
    <el-form :inline="true" :model="state.dataForm" @keyup.enter="state.getDataList()">
      <el-form-item>
        <el-input v-model="state.dataForm.name" placeholder="类别名称" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="state.getDataList()">{{ $t("query") }}</el-button>
      </el-form-item>
      <el-form-item>
        <el-button type="info" @click="state.exportHandle()">{{ $t("export") }}</el-button>
      </el-form-item>
    </el-form>
    <el-table  @expand-change="loadComments" v-loading="state.dataListLoading" :data="state.dataList" border @selection-change="state.dataListSelectionChangeHandle" style="width: 100%">
      <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
      <el-table-column type="expand">
        <template #default="props">
          <el-table :data="commentsPage">
            <el-table-column header-align="center" align="center" label="职评活动" prop="name" />
            <el-table-column header-align="center" align="center" label="评审进度">
              <template v-slot="scope">
                已投：{{scope.row.voted}}，共{{scope.row.judgeNumber}}人
<!--                <el-progress :percentage="Math.floor((scope.row.voted)/(scope.row.judgeNumber)*100)" />-->
              </template>
            </el-table-column>
            <el-table-column :label="$t('handle')" fixed="right" header-align="center" align="center" width="150">
              <template v-slot="scope">
               <el-button v-if="state.hasPermission('occupation:categorie:checkResult') && scope.row.status != 0" type="primary" link @click="checkResultHandle(props.row.id,scope.row.id,scope.row.isPass)">查看{{scope.row.status == 1 ? '过程':'结果'}}</el-button>
              </template>
            </el-table-column>
          </el-table>
        </template>
      </el-table-column>
      <el-table-column prop="name" label="类别名称" header-align="center" align="center"></el-table-column>
      <el-table-column prop="info" label="类别简介" header-align="center" align="center"></el-table-column>
    </el-table>
    <el-pagination :current-page="state.page" :page-sizes="[10, 20, 50, 100]" :page-size="state.limit" :total="state.total" layout="total, sizes, prev, pager, next, jumper" @size-change="state.pageSizeChangeHandle" @current-change="state.pageCurrentChangeHandle"> </el-pagination>

    <CheckResult ref="checkResultKey"></CheckResult>
    </el-drawer>
  </div>
</template>

<script lang="ts" setup>
import useView from "@/hooks/useView";
import { nextTick, reactive, ref, toRefs } from "vue";
import CheckResult from "./result.vue"
import baseService from "@/service/baseService";
import Template from "@/views/devtools/template.vue";

const view = reactive({
  getDataListURL: "/occupation/categorie/page",
  getDataListIsPage: true,
  exportURL: "/occupation/categorie/export",
  deleteURL: "/occupation/categorie",
  deleteIsBatch: true,
  dataForm: {
    name: "",
    status: 2,//类别状态
  }
});

const state = reactive({ ...useView(view), ...toRefs(view) });
const drawerHistory = ref(false)
const init = () => {
  drawerHistory.value = true;
  state.getDataList();
};

const commentsPage = ref([]);
const loadComments = (row: Comment, expandedRows:Comment) => {
  console.log(row)
  console.log(expandedRows)
  baseService.get("/occupation/categorie/CommentsPage",{id:row.id}).then(res=>{
    console.log(res)
    if(res.code!=0) return false
    commentsPage.value = res.data.list;
  })

}

const checkResultKey = ref();
const checkResultHandle = (id:string,commentId:string)=>{
  nextTick(() => {
    checkResultKey.value.init(id,commentId);
  });
}

defineExpose({
  init
})
</script>
