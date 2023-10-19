<template>
  <el-dialog v-model="visible" :title="!dataForm.id ? $t('add') : $t('update')" :close-on-click-modal="false" :close-on-press-escape="false">
    <el-form :model="dataForm" :rules="rules" ref="dataFormRef" @keyup.enter="dataFormSubmitHandle()" label-width="120px">
      <el-form-item label="评议活动名称" prop="name">
        <el-input v-model="dataForm.name" placeholder="评议活动名称"></el-input>
      </el-form-item>
      <el-form-item label="评议活动介绍" prop="info">
<!--        <el-input v-model="dataForm.info" placeholder="评议活动介绍"></el-input>-->
        <wangEditor v-model="dataForm.info" style="height: 15vh"></wangEditor>
      </el-form-item>
      <el-form-item v-if="dataForm.stage > 1" label="第几环节" prop="link">
        <!--        <el-input v-model="dataForm.link" placeholder="第几环节"></el-input>-->
        <el-input-number v-model="dataForm.link" :min="0" :max="10" />
      </el-form-item>
      <el-form-item label="第几阶段" prop="stage">
<!--        <el-input v-model="dataForm.stage" placeholder="第几阶段"></el-input>-->
        <el-input-number v-model="dataForm.stage" :min="0" :max="10" />
      </el-form-item>

<!--      <el-form-item label="活动状态" prop="status">-->
<!--        <el-input v-model="dataForm.status" placeholder="活动状态"></el-input>-->
<!--      </el-form-item>-->
<!--      <el-form-item label="上级评审ids" prop="superiors">-->
<!--        <el-radio-group v-model="isZero">-->
<!--          <el-radio label="1">首次</el-radio>-->
<!--          <el-radio label="2">继承以往</el-radio>-->
<!--        </el-radio-group>-->
<!--        <el-input v-if="isZero==='2'" style="width: 50%;margin-left: 5%;" v-model="dataForm.superiors" placeholder="点击选择以往结果"></el-input>-->
<!--      </el-form-item>-->
      <el-form-item v-if="dataForm.stage === 0" label="指标数" prop="indicator">
        <el-input v-model="dataForm.indicator" placeholder="指标数"></el-input>
      </el-form-item>
      <el-form-item label="评审职称等级" prop="position">
<!--        <el-input v-model="dataForm.position" placeholder="职称"></el-input>-->
        <el-select v-model="dataForm.position" filterable placeholder="请选择评审职称等级">
          <el-option v-for="item in positions" :key="item.value" :label="item.label" :value="item.value"/>
        </el-select>
      </el-form-item>
      <el-form-item label="学科组" prop="sGroup">
<!--        <el-input v-model="dataForm.sGroup" placeholder="学科组"></el-input>-->
        <el-select v-model="dataForm.sGroup" filterable placeholder="请选择学科">
          <el-option v-for="item in groups" :key="item.value" :label="item.label" :value="item.value"/>
        </el-select>
      </el-form-item>
      <el-form-item label="推荐类型" prop="recommendType">
<!--        <el-input v-model="dataForm.recommendType" placeholder="推荐类型"></el-input>-->
        <el-select v-model="dataForm.recommendType" filterable placeholder="请选择推荐类型">
          <el-option v-for="item in recommendTypes" :key="item.value" :label="item.label" :value="item.value"/>
        </el-select>
      </el-form-item>
      <el-form-item label="申报类型" prop="applicationType">
<!--        <el-input v-model="dataForm.applicationType" placeholder="申报类型"></el-input>-->
        <el-select v-model="dataForm.applicationType" filterable placeholder="请选择申报类型">
          <el-option v-for="item in applicationTypes" :key="item.value" :label="item.label" :value="item.value"/>
        </el-select>
      </el-form-item>
      <el-form-item v-if="dataForm.stage == 0" label="竞争性参评人数（竞争型总人数）" prop="competitiveNum">
        <el-input v-model="dataForm.competitiveNum" placeholder="竞争性参评人数（竞争型总人数）"></el-input>
      </el-form-item>
      <el-form-item v-if="dataForm.stage == 0"  label="竞争性指标数" prop="competitiveIndicator">
        <el-input v-model="dataForm.competitiveIndicator" placeholder="竞争性指标数"></el-input>
      </el-form-item>
      <el-form-item v-if="dataForm.stage == 0" label="选取人数（参与人数）" prop="participantNum">
        <el-input v-model="dataForm.participantNum" placeholder="选取人数（参与人数）"></el-input>
      </el-form-item>
<!--      <el-form-item label="剩余指标数" prop="residualIndicator">-->
<!--        <el-input v-model="dataForm.residualIndicator" placeholder="剩余指标数"></el-input>-->
<!--      </el-form-item>-->

    </el-form>
    <template v-slot:footer>
      <el-button @click="visible = false">{{ $t("cancel") }}</el-button>
      <el-button type="primary" @click="dataFormSubmitHandle()">{{ $t("confirm") }}</el-button>
    </template>
  </el-dialog>
  <el-dialog v-model="state.addUserVisible" title="用户" :close-on-click-modal="false" :close-on-press-escape="false">
<!--    <Dept :placeholder="$t('dept.title')" v-model="dataForm.deptName" @userForm="userForm" :projectId="props.projectId" :userpage="state.userpage" :userlimit="state.userlimit" :totalUser="state.totalUser" :role="dataForm.role" ></Dept><br>-->
    <el-table v-loading="state.dataListLoading" ref="dataUserListRef" :data="state.dataUserForm" border @selection-change="dataListSelectionChange" style="width: 100%">
      <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
      <el-table-column prop="username" :label="$t('user.username')" sortable="custom" header-align="center" align="center"></el-table-column>
<!--      <el-table-column prop="deptName" :label="$t('user.deptName')" header-align="center" align="center"></el-table-column>-->
      <el-table-column prop="status" :label="$t('user.status')" sortable="custom" header-align="center" align="center">
        <template v-slot="scope">
          <el-tag v-if="scope.row.status === 0" size="small" type="danger">{{ $t("user.status0") }}</el-tag>
          <el-tag v-else size="small" type="success">{{ $t("user.status1") }}</el-tag>
        </template>
      </el-table-column>
    </el-table>
    <template v-slot:footer>
      <el-button @click="state.addUserVisible = false">{{ $t("cancel") }}</el-button>
      <el-button type="primary" @click="dataFormUserSubmitHandle()">{{ $t("confirm") }}</el-button>
    </template>
    <el-pagination :current-page="state.userpage" :page-sizes="[10, 20, 50, 100]" :page-size="state.userlimit" :total="state.totalUser" layout="total, sizes, prev, pager, next, jumper" @size-change="pageSizeChangeUserHandle" @current-change="pageCurrentChangeUserHandle"> </el-pagination>
  </el-dialog>
</template>

<script lang="ts" setup>
import useView from "@/hooks/useView";
import {onBeforeMount, reactive, ref, toRefs} from "vue";
import baseService from "@/service/baseService";
import { useI18n } from "vue-i18n";
import { ElMessage } from "element-plus";
import wangEditor from "@/components/wang-editor/index.vue";


const { t } = useI18n();
const emit = defineEmits(["refreshDataList"]);

const visible = ref(false);
const isZero = ref('1');
const dataFormRef = ref();

const dataForm = reactive({
  id: "",
  name: "",
  info: "",
  status: 0,
  superiors: "",
  superiorsName:"",
  indicator: "",
  position: "",
  sGroup: "",
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

const applicationTypes = reactive([
  {
    value:3,
    label:'无'
  },
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
    value:3,
    label:'无'
  },
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
    value:2,
    label:'无'
  },
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
    value:3,
    label:'无'
  },
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


const view = reactive({
  roleText:[
    // {value:0,label:"项目创建者"},
    {value:1,label:"项目负责人"},
    {value:2,label:"项目成员"},
    {value:3,label:"项目联系人"},
  ],
  addUserVisible:false,
  getDataListIsPage: true,
  totalUser:0,
  userpage:1,
  userlimit:10,
  dataUserForm: [],
  projectName:""
})

const state = reactive({ ...useView(view), ...toRefs(view) });

const getSuperiors = ()=>{
  baseService.get("/occupation/participant/page",{order: state.order, orderField: state.orderField, page: state.getDataListIsPage ? state.page : null, limit: state.getDataListIsPage ? state.limit : null,
    ...{name: "", applicationType: "", unitId: "", sGroup: "", professionalId: "", positionId: "", recommendType: "", honoraryName: "",}})
}
const dataListSelectionChange=(val:any)=>{
  console.log(val)
  for (let i = 0; i < val.length; i++) {
    dataForm.superiors = dataForm.superiors  + val[i].id + ',';
    dataForm.superiorsName = dataForm.superiorsName + val[i].username + '，';
  }
}

const dataFormUserSubmitHandle = ()=> {
  state.addUserVisible = false;
}

const rules = ref({
});

const init = (id?: number) => {
  visible.value = true;
  dataForm.id = "";

  // 重置表单数据
  if (dataFormRef.value) {
    dataFormRef.value.resetFields();
  }

  if (id) {
    getInfo(id);
  }
};

// 获取信息
const getInfo = (id: number) => {
  baseService.get("/occupation/comment/" + id).then((res) => {
    Object.assign(dataForm, res.data);
  });
};

// 表单提交
const dataFormSubmitHandle = () => {
  dataFormRef.value.validate((valid: boolean) => {
    if (!valid) {
      return false;
    }
    (!dataForm.id ? baseService.post : baseService.put)("/occupation/comment", dataForm).then((res) => {
      ElMessage.success({
        message: t("prompt.success"),
        duration: 500,
        onClose: () => {
          visible.value = false;
          emit("refreshDataList");
        }
      });
    });
  });
};

defineExpose({
  init
});
</script>
