<template>
  <el-dialog v-model="visible" :title="!dataForm.id ? $t('add') : $t('update')" :close-on-click-modal="false" :close-on-press-escape="false">
    <el-form :model="dataForm" :rules="rules" ref="dataFormRef" @keyup.enter="dataFormSubmitHandle()" label-width="22%">
      <el-form-item label="评议活动名称" prop="name">
        <el-input v-model="dataForm.name" placeholder="评议活动名称"></el-input>
      </el-form-item>
      <el-form-item label="评议活动介绍" prop="info">
<!--        <el-input v-model="dataForm.info" placeholder="评议活动介绍"></el-input>-->
        <wangEditor v-model="dataForm.info" style="height: 15vh"></wangEditor>
      </el-form-item>
      <el-form-item label="第几环节" prop="link">
        <!--        <el-input v-model="dataForm.link" placeholder="第几环节"></el-input>-->
        <el-input-number v-model="dataForm.link" :min="0" :max="10" />
      </el-form-item>
      <el-form-item v-if="dataForm.link > 1" label="第几阶段" prop="stage">
        <!--        <el-input v-model="dataForm.stage" placeholder="第几阶段"></el-input>-->
        <el-input-number v-model="dataForm.stage" :min="1" :max="2" />(至少第1阶段，至多第2阶段)
      </el-form-item>
      <el-form-item v-if="dataForm.link > 1" label="第几轮" prop="round">
        <!--        <el-input v-model="dataForm.stage" placeholder="第几阶段"></el-input>-->
        <el-input-number v-model="dataForm.round" :min="1" :max="3" />(至少第1轮，至多第3轮)
      </el-form-item>
      <el-form-item label="是否是附加轮" prop="isAdditional">
        <el-radio-group v-model="dataForm.isAdditional">
          <el-radio label="0">否</el-radio>
          <el-radio label="1">是</el-radio>
        </el-radio-group>
      </el-form-item>

<!--      <el-form-item label="活动状态" prop="status">-->
<!--        <el-input v-model="dataForm.status" placeholder="活动状态"></el-input>-->
<!--      </el-form-item>-->
      <el-form-item label="是否继承上级评审ids" prop="superiors">
        <el-radio-group v-model="isZero">
          <el-radio label="1">是</el-radio>
          <el-radio label="2">否</el-radio>
        </el-radio-group>
        <el-select v-if="isZero == '1'" v-model="dataForm.superiorsArr" multiple collapse-tags collapse-tags-tooltip placeholder="选择上级评审" style="width: 73.5%;margin-left: 2vw">
          <el-option v-for="item in overdResId" :key="item.id" :label="item.name" :value="item.id"/>
        </el-select>
      </el-form-item>
      <el-form-item v-if="dataForm.link == 0" label="指标数" prop="indicator">
        <el-input v-model="dataForm.indicator" placeholder="指标数"></el-input>
      </el-form-item>
      <el-form-item label="评审职称等级" prop="position">
<!--        <el-input v-model="dataForm.position" placeholder="职称"></el-input>-->
        <el-select v-model="dataForm.position" filterable placeholder="请选择评审职称等级">
          <el-option v-for="item in positions" :key="item.value" :label="item.label" :value="item.value"/>
        </el-select>
      </el-form-item>
      <el-form-item label="学科组" prop="sgroup">
<!--        <el-input v-model="dataForm.sgroup" placeholder="学科组"></el-input>-->
        <el-select v-model="dataForm.sgroup" filterable placeholder="请选择学科">
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
      <el-form-item v-if="dataForm.link == 0" label="选取人数" prop="participantNum">
        <el-input v-model="dataForm.participantNum" placeholder="选取人数"></el-input>
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
const isZero = ref(0);
const dataFormRef = ref();

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
  round:"",
  isAdditional:0,
  recommendType: "",
  applicationType: "",
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
const overdResId = ref([]);

const rules = ref({
});

const init = (id?: number) => {
  visible.value = true;
  dataForm.id = "";
  baseService.get("/occupation/comment/page",{
    order: "",
    orderField: "",
    page: 1,
    limit: 100,
    ... {name: "", status: 2, position: "", sgroup: "", recommendType: "", applicationType: "",}
  }).then(res=>{
    console.log(res)
    if(res.code!=0) return false
    overdResId.value = res.data.list;
  })
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
    console.log(res.data)
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
