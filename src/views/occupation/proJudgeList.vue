<template>
  <div class="mod-occupation__professor">
    <el-drawer v-model="drawer" title="查看已选专家" direction="rtl" size="95%">
    <el-form :inline="true" :model="state.dataForm" @keyup.enter="getProJudgeList">
      <el-form-item>
        <el-button v-if="state.hasPermission('occupation:procomment:queryProfessor')" type="primary" @click="queryProfessor(state.procommentId)">选择参与专家条件</el-button>
      </el-form-item>
      <el-form-item>
        <el-button v-if="state.hasPermission('occupation:professor:exportProJudgeList')" type="info" @click="exportProJudgeList()">导出</el-button>
      </el-form-item>
    </el-form>
    <el-table v-loading="state.dataListLoading" :data="ProJudgeList" border @selection-change="state.dataListSelectionChangeHandle" style="width: 100%">
      <el-table-column prop="professor.name" label="专家名称" header-align="center" align="center" width="70"></el-table-column>
      <el-table-column prop="professor.staffId" label="职工编号" header-align="center" align="center" width="95"></el-table-column>
      <el-table-column prop="professor.sex" label="性别" header-align="center" align="center" width="40">
        <template v-slot="scope">
          {{ state.getDictLabel("gender", scope.row.sex) }}
        </template>
      </el-table-column>
      <el-table-column prop="professor.birth" label="出生日期" header-align="center" align="center" width="97"></el-table-column>
      <el-table-column prop="professor.age" label="年龄" header-align="center" align="center" width="41"></el-table-column>
      <el-table-column prop="professor.education" label="学历" header-align="center" align="center"></el-table-column>
      <el-table-column prop="professor.degree" label="学位" header-align="center" align="center"></el-table-column>
      <el-table-column prop="professor.subjects" label="学科" header-align="center" align="center"></el-table-column>
      <el-table-column prop="professor.unitName" label="单位" header-align="center" align="center" show-overflow-tooltip></el-table-column>
      <el-table-column prop="professor.occupation" label="职称" header-align="center" align="center"></el-table-column>
      <el-table-column prop="professor.ranks" label="专业技术级别" header-align="center" align="center"></el-table-column>
      <el-table-column prop="professor.officeDate" label="聘任起始日期" header-align="center" align="center" width="97"></el-table-column>
      <el-table-column :label="$t('handle')" fixed="right" header-align="center" align="center" width="150">
        <template v-slot="scope">
          <el-button v-if="state.hasPermission('occupation:professor:delete')" type="primary" link @click="state.deleteHandle(scope.row.id)">{{ $t("delete") }}</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination :current-page="state.page" :page-sizes="[10, 20, 50, 100]" :page-size="state.limit" :total="state.total" layout="total, sizes, prev, pager, next, jumper" @size-change="state.pageSizeChangeHandle" @current-change="state.pageCurrentChangeHandle"> </el-pagination>
    <!-- 选择条件-->
     <QueryProfessor ref="queryProfessorRef" @refreshDataList="state.getDataList"></QueryProfessor>
    </el-drawer>
  </div>
</template>

<script lang="ts" setup>
import useView from "@/hooks/useView";
import { nextTick, reactive, ref, toRefs, watch } from "vue";
import QueryProfessor from "@/views/occupation/queryProfessor.vue";
import baseService from "@/service/baseService";
import app from "@/constants/app";
import qs from "qs";
import {getToken} from "@/utils/cache";

const drawer = ref(false)
const view = reactive({
  // getDataListURL: "/occupation/professor/page",
  getDataListIsPage: true,
  exportURL: "/occupation/professor/export",
  // deleteURL: "/occupation/professor",
  // deleteIsBatch: true,
  procommentId:"",
  dataForm: {
    commentId:"",
    name:"",
    staffId:"",
    sex:"",
    birth:"",
    age:"",
    education:"",
    degree:"",
    subjects:"",
    unitName:"",
    occupation:"",
    ranks:"",
    officeDate:"",
    creator: "",
    createDate: "",
    updator: "",
    updateDate: "",
  }
});

const state = reactive({ ...useView(view), ...toRefs(view) });

const init = (id:string) => {
  drawer.value = true
  state.procommentId = id
  state.dataForm.commentId = id
  getProJudgeList()
}


const ProJudgeList = ref()
const getProJudgeList = ()=> {
  baseService.get("/occupation/commentprofessor/page",{page:state.page,limit:state.limit,commentId:state.procommentId}).then(res=>{
    console.log(res)
    if(res.code != 0) return false
    ProJudgeList.value = res.data.list
  })
}

// 分页, 每页条数
const pageSizeChangeHandle = (val: number) => {
  state.page = 1;
  state.limit = val;
  getProJudgeList();
}
// 分页, 当前页
const pageCurrentChangeHandle = (val: number) => {
  state.page = val;
  getProJudgeList();
}

const queryProfessorRef = ref();
const queryProfessor = (id:string)=>{
  nextTick(()=>{
    queryProfessorRef.value.init(id);
  })
}
// 导出
const exportProJudgeList = () => {
    window.location.href = `${app.api}${state.exportURL}?${qs.stringify({
      ...state.dataForm,
      access_token: getToken()
    })}`;
    // baseService.download(state.exportURL, { ...state.dataForm, token: getToken() });
}

defineExpose({
  init
})

</script>
