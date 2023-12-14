<template>
  <div class="mod-occupation__result">
    <el-drawer v-model="drawerResult"  title="查看结果" direction="rtl" size="85%">
      <el-form :inline="true" :model="state.dataForm" @keyup.enter="state.getDataList()">
<!--        <el-form-item>-->
<!--          <el-input v-model="state.dataForm.commentId" placeholder="评审活动" clearable></el-input>-->
<!--        </el-form-item>-->
<!--        <el-form-item>-->
<!--          <el-input v-model="state.dataForm.participantId" placeholder="参与人" clearable></el-input>-->
<!--        </el-form-item>-->
<!--        <el-form-item>-->
<!--&lt;!&ndash;          <el-input v-model="state.dataForm.result" placeholder="推荐意见（结果）" clearable></el-input>&ndash;&gt;-->
<!--          <el-select v-model="state.dataForm.result" placeholder="推荐意见（结果）">-->
<!--            <el-option v-for="item in resultText" :key="item.value" :label="item.label" :value="item.value"/>-->
<!--          </el-select>-->
<!--        </el-form-item>-->
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
          <el-button type="info" @click="state.getDataList()">刷新</el-button>
        </el-form-item>
<!--        <el-form-item>-->
<!--          <el-button v-if="state.hasPermission('occupation:result:save')" type="primary" @click="addOrUpdateHandle()">{{ $t("add") }}</el-button>-->
<!--        </el-form-item>-->
<!--        <el-form-item>-->
<!--          <el-button v-if="state.hasPermission('occupation:result:delete')" type="danger" @click="state.deleteHandle()">{{ $t("deleteBatch") }}</el-button>-->
<!--        </el-form-item>-->
      </el-form>
      <el-table v-loading="state.dataListLoading" :data="state.dataList" border @selection-change="state.dataListSelectionChangeHandle" style="width: 100%">
        <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
        <el-table-column prop="commentName" label="评审活动" header-align="center" align="center"></el-table-column>
        <el-table-column prop="participantName" label="参与人" header-align="center" align="center"></el-table-column>
<!--        <el-table-column prop="result" label="推荐意见（结果）" header-align="center" align="center"></el-table-column>-->
        <el-table-column prop="count" label="得票数" header-align="center" align="center"></el-table-column>
        <el-table-column :label="$t('handle')" fixed="right" header-align="center" align="center" width="150">
          <template v-slot="scope">
            <!--            <el-button v-if="state.hasPermission('occupation:result:update')" type="primary" link @click="addOrUpdateHandle(scope.row.id)">{{ $t("update") }}</el-button>-->
            <!--            <el-button v-if="state.hasPermission('occupation:result:update')" type="primary" link @click="addOrUpdateHandle(scope.row.id)">{{ $t("update") }}</el-button>-->
            <el-button v-if="state.hasPermission('occupation:result:details')" type="primary" link @click="checkDetails(scope.row.id,scope.row.participantId)">详细</el-button>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination :current-page="state.page" :page-sizes="[10, 20, 50, 100]" :page-size="state.limit" :total="state.total" layout="total, sizes, prev, pager, next, jumper" @size-change="state.pageSizeChangeHandle" @current-change="state.pageCurrentChangeHandle"> </el-pagination>
      <Details ref="checkDetailsRef"></Details>
    </el-drawer>
  </div>
</template>

<script lang="ts" setup>
import useView from "@/hooks/useView";
import { nextTick, reactive, ref, toRefs, watch } from "vue";
import Details from "./details.vue"
import baseService from "@/service/baseService";

const view = reactive({
  getDataListURL: "/occupation/result/page",
  getDataListIsPage: true,
  exportURL: "/occupation/result/export",
  deleteURL: "/occupation/result",
  deleteIsBatch: true,
  commentId:"",
  categoryId:"",
  dataForm: {
    commentId: "",
    participantId: "",
    result: "",
    count: "",
  }
});

const state = reactive({ ...useView(view), ...toRefs(view) });
const drawerResult = ref(false);
const resultList = ref([]);
const resultText = ref([
  {value:0,label:"通过"},
  {value:1,label:"不通过"},
])

const init = (id: string,commentId:string) => {
  drawerResult.value = true;
  state.commentId = commentId;
  state.dataForm.commentId = commentId
  state.categoryId = id
  state.getDataList()
};


const checkDetailsRef = ref()
const checkDetails = (id:string,participantId:string)=> {
  console.log(state.commentId,id)
  nextTick(()=>{
    checkDetailsRef.value.init(id,state.commentId,participantId);
  })
}

defineExpose({
  init
})
</script>
