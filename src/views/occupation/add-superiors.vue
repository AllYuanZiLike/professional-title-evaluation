<template>
  <div class="mod-occupation__participant">
    <el-dialog v-model="superiorAddVisible" title="添加被评审人" width="70%" center>
      <el-form :inline="true" :model="state.dataForm" @keyup.enter="getSuperiors()">
        <!--      <el-form-item>-->
        <!--        <el-input v-model="state.dataForm.name" placeholder="参与人姓名" clearable></el-input>-->
        <!--      </el-form-item>-->
        <el-form-item>
          <!--        <el-input style="width: 160px;" v-model="state.dataForm.honoraryName" placeholder="荣誉名称" clearable></el-input>-->
          <el-select style="width: 160px;" v-model="state.dataForm.honoraryName" clearable placeholder="请选择职称类型">
            <el-option v-for="item in honorTypes" :key="item.value" :label="item.label" :value="item.value"/>
          </el-select>
        </el-form-item>
        <el-form-item>
          <!--        <el-input style="width: 160px;" v-model="state.dataForm.sgroup" placeholder="学科组" clearable></el-input>-->
          <el-select style="width: 160px;" v-model="state.dataForm.sgroup" clearable placeholder="请选择学科组">
            <el-option v-for="item in groups" :key="item.value" :label="item.label" :value="item.value"/>
          </el-select>
        </el-form-item>
        <el-form-item>
          <!--        <el-input style="width: 160px;" v-model="state.dataForm.applicationType" placeholder="申报类型" clearable></el-input>-->
          <el-select style="width: 160px;" v-model="state.dataForm.applicationType" clearable placeholder="请选择申报类型">
            <el-option v-for="item in applicationTypes" :key="item.value" :label="item.label" :value="item.value"/>
          </el-select>
        </el-form-item>
        <!--      <el-form-item>-->
        <!--        <el-input v-model="state.dataForm.unitId" placeholder="推荐单位" clearable></el-input>-->
        <!--      </el-form-item>-->

        <!--      <el-form-item>-->
        <!--        <el-input v-model="state.dataForm.professionalId" placeholder="申报专业" clearable></el-input>-->
        <!--      </el-form-item>-->
        <!--      <el-form-item>-->
        <!--        <el-input v-model="state.dataForm.positionId" placeholder="申报职称" clearable></el-input>-->
        <!--      </el-form-item>-->
        <el-form-item>
          <!--        <el-input style="width: 160px;" v-model="state.dataForm.recommendType" placeholder="推荐类型" clearable></el-input>-->
          <el-select style="width: 160px;" v-model="state.dataForm.recommendType" clearable placeholder="请选择推荐类型">
            <el-option v-for="item in recommendTypes" :key="item.value" :label="item.label" :value="item.value"/>
          </el-select>
        </el-form-item>

        <el-form-item>
          <el-button @click="getSuperiors()">{{ $t("query") }}</el-button>
        </el-form-item>
      </el-form>
      <el-table v-loading="state.dataListLoading" ref="dataUserListRef" :data="state.dataUserForm" border @selection-change="dataListSelectionChange" style="width: 100%">
        <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
        <el-table-column prop="name" :label="$t('user.username')" sortable="custom" header-align="center" align="center"></el-table-column>
        <el-table-column prop="honorary" label="职称类型" header-align="center" align="center"></el-table-column>
        <el-table-column prop="applicationName" label="申报类型" header-align="center" align="center"></el-table-column>
        <el-table-column prop="groupName" label="学科组" header-align="center" align="center"></el-table-column>
        <el-table-column prop="recommendName" label="推荐类型" header-align="center" align="center"></el-table-column>
      </el-table>
      <template v-slot:footer>
        <el-button @click="superiorAddVisible = false">{{ $t("cancel") }}</el-button>
        <el-button type="primary" @click="dataFormUserSubmitHandle()">{{ $t("confirm") }}</el-button>
      </template>
      <el-pagination :current-page="state.page" :page-sizes="[10, 20, 50, 100]" :page-size="state.limit" :total="state.total" layout="total, sizes, prev, pager, next, jumper" @size-change="pageSizeChangeHandle" @current-change="pageCurrentChangeHandle"> </el-pagination>
    </el-dialog>
  </div>
</template>

<script lang="ts" setup>
import useView from "@/hooks/useView";
import { nextTick, reactive, ref, toRefs, watch } from "vue";
import AddOrUpdate from "./participant-add-or-update.vue";
import baseService from "@/service/baseService";
import {ElMessage} from "element-plus";

const view = reactive({
  dataForm: {
    commentId:"",
    superiors: "",
    superiorsArr:[] as any,
    applicationType: "",
    sgroup: "",
    positionId: "",
    recommendType: "",
    honoraryName: "",
  },
  addUserVisible:false,
  total:0,
  page:1,
  limit:10,
  dataUserForm: [],
});

const state = reactive({ ...useView(view), ...toRefs(view) });

const superiorAddVisible = ref(false)

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
    label:"农科"
  },
  {
    value:1,
    label:"自然科学"
  },
  {
    value:2,
    label:"人文社科"
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
const honorTypes = reactive([
  {
    value:0,
    label:'正高'
  },
  {
    value:1,
    label:'副高'
  },
  {
    value:2,
    label:'中级'
  },
])

const init = (id: string) => {
  superiorAddVisible.value = true;
  state.dataForm.commentId = id;
  getSuperiors();
};

const getSuperiors = ()=>{
  baseService.get("/occupation/participant/excludeParticipants",{ page: state.page , limit: state.limit ,commentId:state.dataForm.commentId,...{applicationType:state.dataForm.applicationType,sgroup:state.dataForm.sgroup,recommendType:state.dataForm.recommendType,honoraryName:state.dataForm.honoraryName}}).then(res=>{
    console.log(res)
    if(res.code!=0) return false;
    state.dataUserForm = res.data.list;
    state.total = res.data.total
  })
}

// 分页, 每页条数
const pageSizeChangeHandle = (val: number)=> {
  state.page = 1;
  state.limit = val;
  getSuperiors();
}
// 分页, 当前页
const pageCurrentChangeHandle = (val: number) => {
  state.page = val;
  getSuperiors();
}

const dataListSelectionChange=(val:any)=>{
  console.log(val)
  state.dataForm.superiorsArr = [];
  for (let i = 0; i < val.length; i++) {
    state.dataForm.superiorsArr[i] = val[i].id;
  }
}
const dataForm = reactive({
  id: "",
  name: "",
  info: "",
  status: 0,
  superiors: "",
  superiorsArr:[],
  indicator: "",
  position: "",
  sgroup: "",
  recommendType: "",
  applicationType: "",
  competitiveNum: "",
  competitiveIndicator: "",
  participantNum: "",
  residualIndicator: "",
  stage: "",
  link: "",
  indicatorType: "",
  startTime: "",
  endTime: "",
  delFlag: "",
  creator: "",
  createDate: "",
  updater: "",
  updateDate: "",
  reserve01: "",
  reserve02: "",
  reserve03: "",
});
const emit = defineEmits(['getSuperiorsList'])
const dataFormUserSubmitHandle = ()=> {
  dataForm.superiorsArr = state.dataForm.superiorsArr
  dataForm.id = state.dataForm.commentId
  console.log(dataForm)
  baseService.post("/occupation/comment/addParticipant", {commentId:dataForm.id,participants:dataForm.superiorsArr}).then((res) => {
    console.log(res)
    if(res.code!=0) return false
    ElMessage.success({
      message: "添加成功",
      duration: 500,
      onClose: () => {
        superiorAddVisible.value = false;
        emit("getSuperiorsList")
      }
    });
  })
}

defineExpose({
  init
})

</script>
