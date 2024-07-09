<template>
  <el-dialog v-model="visible" :title="!dataForm.id ? $t('add') : $t('update')" :close-on-click-modal="false" :close-on-press-escape="false">
    <el-form :model="dataForm" :rules="rules" ref="dataFormRef" @keyup.enter="dataFormSubmitHandle()" label-width="120px">
      <el-form-item label="专家名称" prop="name">
        <el-input v-model="dataForm.name" placeholder="专家名称"></el-input>
      </el-form-item>
      <el-form-item label="职工编号" prop="name">
        <el-input v-model="dataForm.staffId" placeholder="职工编号"></el-input>
      </el-form-item>
      <el-form-item label="性别" prop="sex">
        <ren-radio-group v-model="dataForm.sex" dict-type="gender"></ren-radio-group>
      </el-form-item>
      <el-form-item label="出生日期" prop="birth">
        <el-date-picker v-model="dataForm.birth" type="date" placeholder="请选择日期"/>
      </el-form-item>
      <el-form-item label="年龄" prop="age">
        <el-input v-model="dataForm.age" placeholder="年龄"></el-input>
      </el-form-item>
      <el-form-item label="学历" prop="degree">
        <el-input v-model="dataForm.degree" placeholder="学历"></el-input>
      </el-form-item>
      <el-form-item label="学科" prop="subjects">
        <el-input v-model="dataForm.subjects" placeholder="学科"></el-input>
      </el-form-item>
      <el-form-item label="单位" prop="unitName">
        <el-input v-model="dataForm.unitName" placeholder="单位"></el-input>
      </el-form-item>
      <el-form-item label="职称" prop="occupation">
        <el-input v-model="dataForm.occupation" placeholder="职称"></el-input>
      </el-form-item>
      <el-form-item label="专业技术级别" prop="rNKS">
        <el-input v-model="dataForm.ranks" placeholder="专业技术级别"></el-input>
      </el-form-item>
      <el-form-item label="聘任起始日期" prop="officeDate">
        <el-date-picker v-model="dataForm.officeDate" type="date" placeholder="请选择日期"/>
      </el-form-item>
    </el-form>
    <template v-slot:footer>
      <el-button @click="visible = false">{{ $t("cancel") }}</el-button>
      <el-button type="primary" @click="dataFormSubmitHandle()">{{ $t("confirm") }}</el-button>
    </template>
  </el-dialog>
</template>

<script lang="ts" setup>
import { reactive, ref } from "vue";
import baseService from "@/service/baseService";
import { useI18n } from "vue-i18n";
import { ElMessage } from "element-plus";
const { t } = useI18n();
const emit = defineEmits(["refreshDataList"]);

const visible = ref(false);
const dataFormRef = ref();

const dataForm = reactive({
  id: "",
  name:"",
  staffId:"",
  sex:0,
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
});

const rules = ref({
  name: [{ required: true, message: t("validate.required"), trigger: "blur" }],
  unitId: [{ required: true, message: t("validate.required"), trigger: "blur" }],
  sex: [{ required: true, message: t("validate.required"), trigger: "blur" }],
  occupationId: [{ required: true, message: t("validate.required"), trigger: "blur" }],
  teachYear: [{ required: true, message: t("validate.required"), trigger: "blur" }],
  isOffice: [{ required: true, message: t("validate.required"), trigger: "blur" }],
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
  baseService.get("/occupation/professor/" + id).then((res) => {
    Object.assign(dataForm, res.data);
  });
};

// 表单提交
const dataFormSubmitHandle = () => {
  dataFormRef.value.validate((valid: boolean) => {
    if (!valid) {
      return false;
    }
    (!dataForm.id ? baseService.post : baseService.put)("/occupation/professor", dataForm).then((res) => {
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
