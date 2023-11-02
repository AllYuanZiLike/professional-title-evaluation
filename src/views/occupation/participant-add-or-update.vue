<template>
  <el-dialog v-model="visible" :title="!dataForm. id ? $t('add') : $t('update')" :close-on-click-modal="false" :close-on-press-escape="false">
    <el-form :model="dataForm" :rules="rules" ref="dataFormRef" @keyup.enter="dataFormSubmitHandle()" label-width="120px">
      <el-form-item label="参与人姓名" prop="name">
        <el-input v-model="dataForm.name" placeholder="参与人姓名"></el-input>
      </el-form-item>
      <el-form-item label="申报类型" prop="applicationType">
<!--        <el-input v-model="dataForm.applicationType" placeholder="申报类型"></el-input>-->
        <el-select v-model="dataForm.applicationType" placeholder="请选择申报类型">
          <el-option v-for="item in applicationTypes" :key="item.value" :label="item.label" :value="item.value"/>
        </el-select>
      </el-form-item>
      <el-form-item label="推荐单位" prop="unitId">
<!--        <el-input v-model="dataForm.unitId" placeholder="推荐单位"></el-input>-->
        <el-select v-model="dataForm.unitId" filterable placeholder="请选择推荐单位">
          <el-option v-for="item in units" :key="item.id" :label="item.name" :value="item.id"/>
        </el-select>
      </el-form-item>
      <el-form-item label="学科组" prop="sGroup">
<!--        <el-input v-model="dataForm.group" placeholder="学科组"></el-input>-->
        <el-select v-model="dataForm.sgroup" placeholder="请选择学科组">
          <el-option v-for="item in groups" :key="item.value" :label="item.label" :value="item.value"/>
        </el-select>
      </el-form-item>
      <el-form-item label="申报专业" prop="professionalId">
<!--        <el-input v-model="dataForm.professionalId" placeholder="申报专业"></el-input>-->
        <el-select v-model="dataForm.professionalId" filterable placeholder="请选择申报专业">
          <el-option v-for="item in professes" :key="item.id" :label="item.name" :value="item.id"/>
        </el-select>
      </el-form-item>
      <el-form-item label="申报职称" prop="positionId">
<!--        <el-input v-model="dataForm.positionId" placeholder="申报职称"></el-input>-->
        <el-select v-model="dataForm.positionId" filterable placeholder="请选择申报职称">
          <el-option v-for="item in positions" :key="item.id" :label="item.name" :value="item.id"/>
        </el-select>
      </el-form-item>
      <el-form-item label="推荐类型" prop="recommendType">
<!--        <el-input v-model="dataForm.recommendType" placeholder="推荐类型"></el-input>-->
        <el-select v-model="dataForm.recommendType" placeholder="请选择推荐类型">
          <el-option v-for="item in recommendTypes" :key="item.value" :label="item.label" :value="item.value"/>
        </el-select>
      </el-form-item>
      <el-form-item label="荣誉名称" prop="honoraryName">
<!--        <el-input v-model="dataForm.honoraryName" placeholder="荣誉名称"></el-input>-->
        <el-select v-model="dataForm.honoraryName" placeholder="请选择荣誉名称">
          <el-option v-for="item in honorTypes" :key="item.value" :label="item.label" :value="item.value"/>
        </el-select>
      </el-form-item>
      <el-form-item label="文件" prop="fileId">
<!--        <el-input v-model="dataForm.fileId" placeholder="文件"></el-input>-->
        <el-button @click="uploadFile()">点击上传文件</el-button>
      </el-form-item>
    </el-form>
    <template v-slot:footer>
      <el-button @click="visible = false">{{ $t("cancel") }}</el-button>
      <el-button type="primary" @click="dataFormSubmitHandle()">{{ $t("confirm") }}</el-button>
    </template>
  </el-dialog>
  <FileUpload v-if="uploadVisible" ref="uploadRef" @getFileId="getFileId"></FileUpload>
</template>

<script lang="ts" setup>
import {nextTick,onBeforeMount, reactive, ref} from "vue";
import baseService from "@/service/baseService";
import { useI18n } from "vue-i18n";
import { ElMessage } from "element-plus";
import FileUpload from './file-upload.vue'
const { t } = useI18n();
const emit = defineEmits(["refreshDataList"]);

const visible = ref(false);
const uploadVisible = ref(false);
const dataFormRef = ref();

const dataForm = reactive({
  id: "",
  name: "",
  applicationType: "",
  unitId: "",
  sgroup: "",
  professionalId: "",
  positionId: "",
  recommendType: "",
  honoraryName: "",
  fileId: "",
  fileName:"",
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
const rules = ref({
  name: [{ required: true, message: t("validate.required"), trigger: "blur" }],
  applicationType: [{ required: true, message: t("validate.required"), trigger: "blur" }],
  unitId: [{ required: true, message: t("validate.required"), trigger: "blur" }],
  sgroup: [{ required: true, message: t("validate.required"), trigger: "blur" }],
  professionalId: [{ required: true, message: t("validate.required"), trigger: "blur" }],
  positionId: [{ required: true, message: t("validate.required"), trigger: "blur" }],
  recommendType: [{ required: true, message: t("validate.required"), trigger: "blur" }],
  honoraryName: [{ required: true, message: t("validate.required"), trigger: "blur" }],
  fileId: [{ required: true, message: t("validate.required"), trigger: "blur" }],
});
interface ListItem {
  value: string
  label: string
}
let units = ref([])
let professes = ref([])
let positions = ref([])
const getInitData = ()=> {
  baseService.get("/occupation/unit/page", {order: "", orderField: "", page: 1, limit: 100, ...{name: "",}}).then(res=>{
    if(res.code !== 0) return false;
    units.value = res.data.list;
    console.log(units.value)
  })
  baseService.get("/occupation/professional/page", {order: "", orderField: "", page: 1, limit: 100, ...{name: "",}}).then(res=>{
    if(res.code !== 0) return false;
    professes.value = res.data.list;
  })
  baseService.get("/occupation/poition/page", {order: "", orderField: "", page: 1, limit: 100, ...{name: "",}}).then(res=>{
    if(res.code !== 0) return false;
    positions.value = res.data.list;
  })
}
const init = ( id?: number) => {
  visible.value = true;
  dataForm.id = "";
  // 重置表单数据
  if (dataFormRef.value) {
    dataFormRef.value.resetFields();
  }

  if ( id) {
    getInfo( id);
  }
};

// 获取信息
const getInfo = ( id: number) => {
  baseService.get("/occupation/participant/" +  id).then((res) => {
    console.log(res)
    Object.assign(dataForm, res.data);
  });

};

//上传文件
const uploadRef = ref()
const uploadFile = ()=>{
  uploadVisible.value = true;
  nextTick(() => {
    uploadRef.value.init();
  });
}
const getFileId = (list:any)=>{
  console.log(list.value[0].name)
  console.log(list.value[0].response.data.url)
  // dataForm.fileId = list.value[0].uid.toString();
  dataForm.fileName = list.value[0].name;
  dataForm.reserve01 = list.value[0].response.data.url;
}
// 表单提交
const dataFormSubmitHandle = () => {
  console.log(dataForm)
  dataFormRef.value.validate((valid: boolean) => {
    if (!valid) {
      return false;
    }
    (!dataForm. id ? baseService.post : baseService.put)("/occupation/participant", dataForm).then((res) => {
      if(res.code!==0) return false;
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
onBeforeMount(()=>{
  getInitData()
})
defineExpose({
  init
});
</script>
