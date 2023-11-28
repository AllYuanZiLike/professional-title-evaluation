<template>
  <div class="mod-occupation__comment">
    <el-drawer v-model="drawerSuper"  title="被评审人列表" direction="rtl" size="85%">
      <el-form :inline="true" :model="state.dataForm" @keyup.enter="getParticipants">
        <el-form-item style="width: 8vw;">
          <el-select v-model="state.dataForm.honoraryName" clearable filterable placeholder="职称类型">
            <el-option v-for="item in positions" :key="item.value" :label="item.label" :value="item.value"/>
          </el-select>
        </el-form-item>
        <el-form-item style="width: 8vw;">
          <!--        <el-input v-model="state.dataForm.sgroup" placeholder="学科组" clearable></el-input>-->
          <el-select v-model="state.dataForm.sgroup" clearable filterable placeholder="学科组">
            <el-option v-for="item in groups" :key="item.value" :label="item.label" :value="item.value"/>
          </el-select>
        </el-form-item>
        <el-form-item style="width: 8vw;">
          <!--        <el-input v-model="state.dataForm.recommendType" placeholder="推荐类型" clearable></el-input>-->
          <el-select v-model="state.dataForm.recommendType" clearable filterable placeholder="推荐类型">
            <el-option v-for="item in recommendTypes" :key="item.value" :label="item.label" :value="item.value"/>
          </el-select>
        </el-form-item>
        <el-form-item style="width: 8vw;">
          <!--        <el-input v-model="state.dataForm.applicationType" placeholder="申报类型" clearable></el-input>-->
          <el-select v-model="state.dataForm.applicationType" clearable filterable placeholder="申报类型">
            <el-option v-for="item in applicationTypes" :key="item.value" :label="item.label" :value="item.value"/>
          </el-select>
        </el-form-item>
        <el-form-item>
          <el-button @click="getParticipants()">{{ $t("query") }}</el-button>
        </el-form-item>
        <el-form-item>
          <el-button v-if="state.hasPermission('occupation:comment:save')" type="primary" @click="superiorAddHandle(state.commentId)">{{ $t("add") }}</el-button>
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
        <el-table-column prop="name" :label="$t('user.username')" sortable="custom" header-align="center" align="center"></el-table-column>
        <el-table-column prop="honorary" label="职称类型" header-align="center" align="center"></el-table-column>
        <el-table-column prop="applicationName" label="申报类型" header-align="center" align="center"></el-table-column>
        <el-table-column prop="groupName" label="学科组" header-align="center" align="center"></el-table-column>
        <el-table-column prop="recommendName" label="推荐类型" header-align="center" align="center"></el-table-column>
        <el-table-column :label="$t('handle')" fixed="right" header-align="center" align="center" width="150">
          <template v-slot="scope">
            <el-button v-if="state.hasPermission('occupation:comment:delete')" type="primary" link @click="delSuperior(scope.row.id)">{{ $t("delete") }}</el-button>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination :current-page="state.page" :page-sizes="[10, 20, 50, 100]" :page-size="state.limit" :total="state.total" layout="total, sizes, prev, pager, next, jumper" @size-change="pageSizeChangeHandle" @current-change="pageCurrentChangeHandle"> </el-pagination>
      <!-- 弹窗, 新增  -->
      <SuperiorAdd ref="superiorAddRef" @getSuperiorsList="getParticipants()"></SuperiorAdd>
    </el-drawer>
  </div>
</template>

<script lang="ts" setup>
import useView from "@/hooks/useView";
import { nextTick, reactive, ref, toRefs, watch } from "vue";
import SuperiorAdd from "./add-superiors.vue";
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
const drawerSuper = ref(false);
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
  drawerSuper.value = true;
  state.commentId = id;
  getParticipants()
};

const getParticipants = ()=>{
  baseService.get("/occupation/participant/getParticipants",{page: state.page, limit:  state.limit,commentId:state.commentId,...{applicationType:state.dataForm.applicationType,sgroup:state.dataForm.sgroup,recommendType:state.dataForm.recommendType,honoraryName:state.dataForm.honoraryName}}).then(res=>{
    console.log(res)
    if(res.code != 0) return false
    participants.value = res.data.list;
    state.total = res.data.total;
  })
}

// 分页, 每页条数
const pageSizeChangeHandle = (val: number)=> {
  state.page = 1;
  state.limit = val;
  getParticipants();
}
// 分页, 当前页
const pageCurrentChangeHandle = (val: number) => {
  state.page = val;
  getParticipants();
}

const delSuperiorIds = ref<Array<string>>([])
const delSuperior = (id? :string) => {
  delSuperiorIds.value = [];
    if(state.dataListSelections){
      state.dataListSelections.map((item: IObject) => {
        delSuperiorIds.value.push(item.id)
      })
  }
  if(id) delSuperiorIds.value.push(id)
  console.log(delSuperiorIds.value)
  console.log(state.commentId)
  ElMessageBox.confirm("确定进行删除操作？","提示", {
    confirmButtonText:"确定",
    cancelButtonText: "取消",
    type: "warning"
  }).then(() => {
      baseService.post("/occupation/comment/delParticipant",{ids:delSuperiorIds.value,commentId:state.commentId}).then(res=>{
        console.log(res)
        if(res.code != 0) return false
        ElMessage.success("删除成功")
        getParticipants()
      })
    })
}
const superiorAddRef = ref();
const superiorAddHandle = (id: string) => {
  nextTick(() => {
    superiorAddRef.value.init(id);
  });
};

defineExpose({
  init
})
</script>
