<template>
  <el-dialog v-model="visible" :title="$t('oss.config')" :close-on-click-modal="false" :close-on-press-escape="false">
    <el-form :model="dataForm" :rules="rules" ref="dataFormRef" @keyup.enter="dataFormSubmitHandle()" label-width="140px">
      <el-form-item :label="$t('oss.type')">
        <el-radio-group v-model="dataForm.type">
          <el-radio :label="1">{{ $t("oss.type1") }}</el-radio>
          <el-radio :label="2">{{ $t("oss.type2") }}</el-radio>
          <el-radio :label="3">{{ $t("oss.type3") }}</el-radio>
          <el-radio :label="4">{{ $t("oss.type4") }}</el-radio>
          <el-radio :label="5">{{ $t("oss.type5") }}</el-radio>
          <el-radio :label="6">{{ $t("oss.type6") }}</el-radio>
        </el-radio-group>
      </el-form-item>
      <template v-if="dataForm.type === 1">
        <el-form-item prop="qiniuDomain" :label="$t('oss.qiniuDomain')">
          <el-input v-model="dataForm.qiniuDomain" :placeholder="$t('oss.qiniuDomainTips')"></el-input>
        </el-form-item>
        <el-form-item prop="qiniuPrefix" :label="$t('oss.qiniuPrefix')">
          <el-input v-model="dataForm.qiniuPrefix" :placeholder="$t('oss.qiniuPrefixTips')"></el-input>
        </el-form-item>
        <el-form-item prop="qiniuAccessKey" :label="$t('oss.qiniuAccessKey')">
          <el-input v-model="dataForm.qiniuAccessKey" :placeholder="$t('oss.qiniuAccessKeyTips')"></el-input>
        </el-form-item>
        <el-form-item prop="qiniuSecretKey" :label="$t('oss.qiniuSecretKey')">
          <el-input v-model="dataForm.qiniuSecretKey" :placeholder="$t('oss.qiniuSecretKeyTips')"></el-input>
        </el-form-item>
        <el-form-item prop="qiniuBucketName" :label="$t('oss.qiniuBucketName')">
          <el-input v-model="dataForm.qiniuBucketName" :placeholder="$t('oss.qiniuBucketNameTips')"></el-input>
        </el-form-item>
      </template>
      <template v-else-if="dataForm.type === 2">
        <el-form-item prop="aliyunDomain" :label="$t('oss.aliyunDomain')">
          <el-input v-model="dataForm.aliyunDomain" :placeholder="$t('oss.aliyunDomainTips')"></el-input>
        </el-form-item>
        <el-form-item prop="aliyunPrefix" :label="$t('oss.aliyunPrefix')">
          <el-input v-model="dataForm.aliyunPrefix" :placeholder="$t('oss.aliyunPrefixTips')"></el-input>
        </el-form-item>
        <el-form-item prop="aliyunEndPoint" :label="$t('oss.aliyunEndPoint')">
          <el-input v-model="dataForm.aliyunEndPoint" :placeholder="$t('oss.aliyunEndPointTips')"></el-input>
        </el-form-item>
        <el-form-item prop="aliyunAccessKeyId" :label="$t('oss.aliyunAccessKeyId')">
          <el-input v-model="dataForm.aliyunAccessKeyId" :placeholder="$t('oss.aliyunAccessKeyIdTips')"></el-input>
        </el-form-item>
        <el-form-item prop="aliyunAccessKeySecret" :label="$t('oss.aliyunAccessKeySecret')">
          <el-input v-model="dataForm.aliyunAccessKeySecret" :placeholder="$t('oss.aliyunAccessKeySecretTips')"></el-input>
        </el-form-item>
        <el-form-item prop="aliyunBucketName" :label="$t('oss.aliyunBucketName')">
          <el-input v-model="dataForm.aliyunBucketName" :placeholder="$t('oss.aliyunBucketNameTips')"></el-input>
        </el-form-item>
      </template>
      <template v-else-if="dataForm.type === 3">
        <el-form-item prop="qcloudDomain" :label="$t('oss.qcloudDomain')">
          <el-input v-model="dataForm.qcloudDomain" :placeholder="$t('oss.qcloudDomainTips')"></el-input>
        </el-form-item>
        <el-form-item prop="qcloudPrefix" :label="$t('oss.qcloudPrefix')">
          <el-input v-model="dataForm.qcloudPrefix" :placeholder="$t('oss.qcloudPrefixTips')"></el-input>
        </el-form-item>
        <el-form-item prop="qcloudAppId" :label="$t('oss.qcloudAppId')">
          <el-input v-model="dataForm.qcloudAppId" :placeholder="$t('oss.qcloudAppIdTips')"></el-input>
        </el-form-item>
        <el-form-item prop="qcloudSecretId" :label="$t('oss.qcloudSecretId')">
          <el-input v-model="dataForm.qcloudSecretId" :placeholder="$t('oss.qcloudSecretIdTips')"></el-input>
        </el-form-item>
        <el-form-item prop="qcloudSecretKey" :label="$t('oss.qcloudSecretKey')">
          <el-input v-model="dataForm.qcloudSecretKey" :placeholder="$t('oss.qcloudSecretKeyTips')"></el-input>
        </el-form-item>
        <el-form-item prop="qcloudBucketName" :label="$t('oss.qcloudBucketName')">
          <el-input v-model="dataForm.qcloudBucketName" :placeholder="$t('oss.qcloudBucketNameTips')"></el-input>
        </el-form-item>
        <el-form-item prop="qcloudRegion" :label="$t('oss.qcloudRegion')">
          <el-select v-model="dataForm.qcloudRegion" clearable :placeholder="$t('oss.qcloudRegionTips')" class="w-percent-100">
            <el-option value="ap-beijing-1" :label="$t('oss.qcloudRegionBeijing1')"></el-option>
            <el-option value="ap-beijing" :label="$t('oss.qcloudRegionBeijing')"></el-option>
            <el-option value="ap-shanghai" :label="$t('oss.qcloudRegionShanghai')"></el-option>
            <el-option value="ap-guangzhou" :label="$t('oss.qcloudRegionGuangzhou')"></el-option>
            <el-option value="ap-chengdu" :label="$t('oss.qcloudRegionChengdu')"></el-option>
            <el-option value="ap-chongqing" :label="$t('oss.qcloudRegionChongqing')"></el-option>
            <el-option value="ap-singapore" :label="$t('oss.qcloudRegionSingapore')"></el-option>
            <el-option value="ap-hongkong" :label="$t('oss.qcloudRegionHongkong')"></el-option>
            <el-option value="na-toronto" :label="$t('oss.qcloudRegionToronto')"></el-option>
            <el-option value="eu-frankfurt" :label="$t('oss.qcloudRegionFrankfurt')"></el-option>
          </el-select>
        </el-form-item>
      </template>
      <template v-else-if="dataForm.type === 4">
        <el-form-item prop="fastdfsDomain" :label="$t('oss.fastdfsDomain')">
          <el-input v-model="dataForm.fastdfsDomain" :placeholder="$t('oss.fastdfsDomainTips')"></el-input>
        </el-form-item>
      </template>
      <template v-else-if="dataForm.type === 5">
        <el-form-item prop="localDomain" :label="$t('oss.localDomain')">
          <el-input v-model="dataForm.localDomain" :placeholder="$t('oss.localDomainTips')"></el-input>
        </el-form-item>
        <el-form-item prop="localPrefix" :label="$t('oss.localPrefix')">
          <el-input v-model="dataForm.localPrefix" :placeholder="$t('oss.localPrefixTips')"></el-input>
        </el-form-item>
        <el-form-item prop="localPath" :label="$t('oss.localPath')">
          <el-input v-model="dataForm.localPath" :placeholder="$t('oss.localPathTips')"></el-input>
        </el-form-item>
      </template>
      <template v-else-if="dataForm.type === 6">
        <el-form-item prop="minioEndPoint" :label="$t('oss.minioEndPoint')">
          <el-input v-model="dataForm.minioEndPoint" :placeholder="$t('oss.minioEndPointTips')"></el-input>
        </el-form-item>
        <el-form-item prop="minioAccessKey" :label="$t('oss.minioAccessKey')">
          <el-input v-model="dataForm.minioAccessKey" :placeholder="$t('oss.minioAccessKeyTips')"></el-input>
        </el-form-item>
        <el-form-item prop="minioSecretKey" :label="$t('oss.minioSecretKey')">
          <el-input v-model="dataForm.minioSecretKey" :placeholder="$t('oss.minioSecretKeyTips')"></el-input>
        </el-form-item>
        <el-form-item prop="minioBucketName" :label="$t('oss.minioBucketName')">
          <el-input v-model="dataForm.minioBucketName" :placeholder="$t('oss.minioBucketNameTips')"></el-input>
        </el-form-item>
        <el-form-item prop="minioPrefix" :label="$t('oss.minioPrefix')">
          <el-input v-model="dataForm.minioPrefix" :placeholder="$t('oss.minioPrefixTips')"></el-input>
        </el-form-item>
      </template>
    </el-form>
    <template v-slot:footer>
      <el-button @click="visible = false">{{ $t("cancel") }}</el-button>
      <el-button type="primary" @click="dataFormSubmitHandle()">{{ $t("confirm") }}</el-button>
    </template>
  </el-dialog>
</template>

<script lang="ts" setup>
import { reactive, ref, watch } from "vue";
import baseService from "@/service/baseService";
import { useI18n } from "vue-i18n";
import { ElMessage } from "element-plus";
const { t } = useI18n();
const emit = defineEmits(["refreshDataList"]);

const visible = ref(false);
const dataFormRef = ref();

const dataForm = reactive({
  type: 0,
  qiniuDomain: "",
  qiniuPrefix: "",
  qiniuAccessKey: "",
  qiniuSecretKey: "",
  qiniuBucketName: "",
  aliyunDomain: "",
  aliyunPrefix: "",
  aliyunEndPoint: "",
  aliyunAccessKeyId: "",
  aliyunAccessKeySecret: "",
  aliyunBucketName: "",
  qcloudDomain: "",
  qcloudPrefix: "",
  qcloudAppId: 0,
  qcloudSecretId: "",
  qcloudSecretKey: "",
  qcloudBucketName: "",
  qcloudRegion: "",
  fastdfsDomain: "",
  localDomain: "",
  localPrefix: "",
  localPath: "",
  minioPrefix: "",
  minioBucketName: "",
  minioSecretKey: "",
  minioAccessKey: "",
  minioEndPoint: ""
});

const rules = ref({
  qiniuDomain: [{ required: true, message: t("validate.required"), trigger: "blur" }],
  qiniuAccessKey: [{ required: true, message: t("validate.required"), trigger: "blur" }],
  qiniuSecretKey: [{ required: true, message: t("validate.required"), trigger: "blur" }],
  qiniuBucketName: [{ required: true, message: t("validate.required"), trigger: "blur" }],
  aliyunDomain: [{ required: true, message: t("validate.required"), trigger: "blur" }],
  aliyunEndPoint: [{ required: true, message: t("validate.required"), trigger: "blur" }],
  aliyunAccessKeyId: [{ required: true, message: t("validate.required"), trigger: "blur" }],
  aliyunAccessKeySecret: [{ required: true, message: t("validate.required"), trigger: "blur" }],
  aliyunBucketName: [{ required: true, message: t("validate.required"), trigger: "blur" }],
  qcloudDomain: [{ required: true, message: t("validate.required"), trigger: "blur" }],
  qcloudAppId: [{ required: true, message: t("validate.required"), trigger: "blur" }],
  qcloudSecretId: [{ required: true, message: t("validate.required"), trigger: "blur" }],
  qcloudSecretKey: [{ required: true, message: t("validate.required"), trigger: "blur" }],
  qcloudBucketName: [{ required: true, message: t("validate.required"), trigger: "blur" }],
  qcloudRegion: [{ required: true, message: t("validate.required"), trigger: "blur" }],
  fastdfsDomain: [{ required: true, message: t("validate.required"), trigger: "blur" }],
  localDomain: [{ required: true, message: t("validate.required"), trigger: "blur" }],
  localPath: [{ required: true, message: t("validate.required"), trigger: "blur" }],
  minioEndPoint: [{ required: true, message: t("validate.required"), trigger: "blur" }],
  minioAccessKey: [{ required: true, message: t("validate.required"), trigger: "blur" }],
  minioSecretKey: [{ required: true, message: t("validate.required"), trigger: "blur" }],
  minioBucketName: [{ required: true, message: t("validate.required"), trigger: "blur" }]
});

const init = () => {
  visible.value = true;

  // 重置表单数据
  if (dataFormRef.value) {
    dataFormRef.value.resetFields();
  }

  getInfo();
};

// 获取信息
const getInfo = () => {
  baseService.get("/oss/file/info").then((res) => {
    Object.assign(dataForm, res.data);
  });
};

// 表单提交
const dataFormSubmitHandle = () => {
  dataFormRef.value.validate((valid: boolean) => {
    if (!valid) {
      return false;
    }
    baseService.post("/oss/file", dataForm).then((res) => {
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
