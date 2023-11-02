<template>
  <div class="mod-occupation__comment">
    <el-drawer v-model="drawerJudge"  title="被评审人列表" direction="rtl" size="85%">
      <el-form :inline="true" :model="state.dataForm" @keyup.enter="getParticipants">
       <el-form-item style="width: 18vw;">
          <!--        <el-input v-model="state.dataForm.sgroup" placeholder="学科组" clearable></el-input>-->
          <el-select v-model="state.dataForm.sgroup" clearable filterable placeholder="学科组">
            <el-option v-for="item in groups" :key="item.value" :label="item.label" :value="item.value"/>
          </el-select>
        </el-form-item>
        <el-form-item>
          <el-button @click="getParticipants()">{{ $t("query") }}</el-button>
        </el-form-item>
        <el-form-item>
          <el-button v-if="state.hasPermission('occupation:comment:save')" type="primary" @click="judgeAddHandle(state.commentId)">{{ $t("add") }}</el-button>
        </el-form-item>
        <el-form-item>
          <el-button v-if="state.hasPermission('occupation:comment:delete')" type="danger" @click="delSuperior()">{{ $t("deleteBatch") }}</el-button>
        </el-form-item>
        <el-form-item>
          <el-button type="info" @click="getParticipants">刷新</el-button>
        </el-form-item>
      </el-form>
      <el-table v-loading="state.dataListLoading" :data="participants" border @selection-change="state.dataListSelectionChangeHandle" style="width: 100%">
        <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
        <el-table-column prop="username" :label="$t('user.username')" sortable="custom" header-align="center" align="center"></el-table-column>
        <el-table-column prop="deptName" label="学科组" header-align="center" align="center"></el-table-column>
        <el-table-column :label="$t('handle')" fixed="right" header-align="center" align="center" width="150">
          <template v-slot="scope">
            <el-button v-if="state.hasPermission('occupation:comment:delete')" type="primary" link @click="delJudge(scope.row.id)">{{ $t("delete") }}</el-button>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination :current-page="state.page" :page-sizes="[10, 20, 50, 100]" :page-size="state.limit" :total="state.total" layout="total, sizes, prev, pager, next, jumper" @size-change="state.pageSizeChangeHandle" @current-change="state.pageCurrentChangeHandle"> </el-pagination>
      <!-- 弹窗, 新增  -->
      <judgeAdd ref="judgeAddRef" @getSuperiorsList="getParticipants()"></judgeAdd>
    </el-drawer>
  </div>
</template>

<script lang="ts" setup>
import useView from "@/hooks/useView";
import { nextTick, reactive, ref, toRefs, watch } from "vue";
import judgeAdd from "./judge-add-or-update.vue";
import baseService from "@/service/baseService";
import {IObject} from "@/types/interface";
import {ElMessage, ElMessageBox} from "element-plus";

const view = reactive({
  deleteIsBatchKey: "id",
  deleteIsBatch: true,
  commentId:"",
  page:1,
  limit:10,
  dataForm: {
    commentId:"",
    superiors: "",
    superiorsName:"",
    applicationType: "",
    sgroup: "",
    positionId: "",
    recommendType: "",
    honoraryName: "",
  },
});

const state = reactive({ ...useView(view), ...toRefs(view) });
const drawerJudge = ref(false);
const participants = ref([]);
const statusText = reactive([
  {
    value:0,
    label:'未开始'
  },
  {
    value:1,
    label:'进行中'
  },
  {
    value:2,
    label:'已结束'
  },
])
const applicationTypes = reactive([
  {
    value:0,
    label:"教学为主"
  },
  {
    value:1,
    label:"教学科研"
  },
  {
    value:2,
    label:"其他（实验人员）"
  },

])
const groups = reactive([
  {
    value:0,
    label:"自然科学"
  },
  {
    value:1,
    label:"人文社科"
  },
  {
    value:2,
    label:"农科"
  },

])
const recommendTypes = reactive([
  {
    value:0,
    label:'直推'
  },
  {
    value:1,
    label:'竞争'
  },

])
const positions = reactive([
  {
    value:0,
    label:'正高级'
  },
  {
    value:1,
    label:'副高级'
  },
  {
    value:2,
    label:'中级'
  },

])

const init = (id: string) => {
  drawerJudge.value = true;
  state.commentId = id;
  getParticipants()
};

const getParticipants = ()=>{
  baseService.get("/occupation/judge/judgePage",{page: state.page, limit:  state.limit,commentId:state.commentId,...{applicationType:state.dataForm.applicationType,sgroup:state.dataForm.sgroup,recommendType:state.dataForm.recommendType,honoraryName:state.dataForm.honoraryName}}).then(res=>{
    console.log(res)
    if(res.code != 0) return false
    participants.value = res.data.list;
  })
}

const delJudgeIds = ref<Array<string>>([])
const delJudge = (id? :string) => {
  delJudgeIds.value = [];
  if(state.dataListSelections){
    state.dataListSelections.map((item: IObject) => {
      delJudgeIds.value.push(item.id)
    })
  }
  if(id) delJudgeIds.value.push(id)
  console.log(delJudgeIds.value)
  console.log(state.commentId)
  ElMessageBox.confirm("确定进行删除操作？","提示", {
    confirmButtonText:"确定",
    cancelButtonText: "取消",
    type: "warning"
  }).then(() => {
    baseService.post("/occupation/comment/delJudges",{ids:delJudgeIds.value,commentId:state.commentId}).then(res=>{
      console.log(res)
      if(res.code != 0) return false
      ElMessage.success("删除成功")
      getParticipants()
    })
  })
}
const judgeAddRef = ref();
const judgeAddHandle = (id: string) => {
  nextTick(() => {
    judgeAddRef.value.init(id);
  });
};

defineExpose({
  init
})
</script>
