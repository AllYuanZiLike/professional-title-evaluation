<template>
  <div class="mod-occupation__result">
    <el-dialog v-model="drawerResult"  title="详细" width="80%">
      <el-form :inline="true" :model="state.dataForm" @keyup.enter="getDetailsList">
        <!--        <el-form-item>-->
        <!--          <el-input v-model="state.dataForm.commentId" placeholder="评审活动" clearable></el-input>-->
        <!--        </el-form-item>-->
        <!--        <el-form-item>-->
        <!--          <el-input v-model="state.dataForm.participantId" placeholder="参与人" clearable></el-input>-->
        <!--        </el-form-item>-->
<!--        <el-form-item>-->
<!--          &lt;!&ndash;          <el-input v-model="state.dataForm.result" placeholder="推荐意见（结果）" clearable></el-input>&ndash;&gt;-->
<!--          <el-select v-model="state.dataForm.result" placeholder="推荐意见（结果）">-->
<!--            <el-option v-for="item in resultText" :key="item.value" :label="item.label" :value="item.value"/>-->
<!--          </el-select>-->
<!--        </el-form-item>-->
<!--        <el-form-item>-->
<!--          <el-button @click="state.getDataList()">{{ $t("query") }}</el-button>-->
<!--        </el-form-item>-->
        <el-form-item>
          <el-button type="info" @click="state.exportHandle()">{{ $t("export") }}</el-button>
        </el-form-item>
        <!--        <el-form-item>-->
        <!--          <el-button v-if="state.hasPermission('occupation:result:save')" type="primary" @click="addOrUpdateHandle()">{{ $t("add") }}</el-button>-->
        <!--        </el-form-item>-->
        <!--        <el-form-item>-->
        <!--          <el-button v-if="state.hasPermission('occupation:result:delete')" type="danger" @click="state.deleteHandle()">{{ $t("deleteBatch") }}</el-button>-->
        <!--        </el-form-item>-->
      </el-form>
      <el-table v-loading="state.dataListLoading" :data="resultList" border @selection-change="state.dataListSelectionChangeHandle" style="width: 100%">
<!--        <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>-->
        <el-table-column prop="commentName" label="评审活动" header-align="center" align="center"></el-table-column>
        <el-table-column prop="participantName" label="候选人" header-align="center" align="center"></el-table-column>
        <el-table-column prop="opinionName" label="推荐意见" header-align="center" align="center"></el-table-column>
        <el-table-column prop="judgeName" label="评委" header-align="center" align="center"></el-table-column>
        <el-table-column prop="updateDate" label="时间" header-align="center" align="center"></el-table-column>
      </el-table>
<!--      <el-pagination :current-page="state.page" :page-sizes="[10, 20, 50, 100]" :page-size="state.limit" :total="state.total" layout="total, sizes, prev, pager, next, jumper" @size-change="state.pageSizeChangeHandle" @current-change="state.pageCurrentChangeHandle"> </el-pagination>-->
    </el-dialog>
  </div>
</template>

<script lang="ts" setup>
import useView from "@/hooks/useView";
import { nextTick, reactive, ref, toRefs, watch } from "vue";
import baseService from "@/service/baseService";
import dayjs from "dayjs";

const view = reactive({
  getDataListURL: "/occupation/result/page",
  getDataListIsPage: true,
  exportURL: "/occupation/count/export",
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
const resultList = ref([{updateDate:'' as any}]);
const resultText = ref([
  {value:0,label:"通过"},
  {value:1,label:"不通过"},
])

const init = (id: string,commentId:string,participantId:string) => {
  state.commentId = commentId;
  state.dataForm.commentId = commentId
  state.dataForm.participantId = participantId
  state.categoryId = id
  getDetailsList()
  drawerResult.value = true;
};

const getDetailsList = ()=>{
  baseService.post("/occupation/result/show",{commentId:state.commentId,participantId:state.dataForm.participantId}).then(res=>{
    console.log(res)
    if(res.code != 0) return false
    resultList.value = res.data.list;
    state.total = res.data.total;
    resultList.value.map(item=>{
      let a = dayjs(item.updateDate)
      item.updateDate = a.format('YYYY年MM月DD日 HH:mm:ss')
      console.log(item.updateDate)
    })
  })
}

defineExpose({
  init
})
</script>
