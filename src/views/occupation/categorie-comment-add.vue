<template>
  <el-dialog v-model="visible" title="添加职评活动" :close-on-click-modal="false" :close-on-press-escape="false" width="30%">
    <el-select v-model="commentId" placeholder="请选择职评活动" multiple style="width: 100%;">
      <el-option v-for="item in commentList" :key="item.id" :label="item.name" :value="item.id"/>
    </el-select>
    <template v-slot:footer>
      <el-button @click="visible = false">{{ $t("cancel") }}</el-button>
      <el-button type="primary" @click="submitCommentId()">{{ $t("confirm") }}</el-button>
    </template>
  </el-dialog>
</template>

<script lang="ts" setup>
import { ref } from "vue";
import { useI18n } from "vue-i18n";
import { ElMessage } from "element-plus";
import baseService from "@/service/baseService";

const visible = ref(false);
const commentList = ref([]);
const commentId = ref([]);
const categoryId = ref()

const init = (id:string) => {
  categoryId.value = id
  baseService.get("/occupation/comment/page",{order: "", orderField: "", page: 1, limit: 100, ...{name: "", status: 0, position: "", sgroup: "", recommendType: "", applicationType: "",}
  }).then(res=>{
    console.log(res)
    if(res.code!=0) return false
    commentList.value=res.data.list;
  })
  visible.value = true;
};

const submitCommentId = ()=>{
  console.log(commentId.value)
  baseService.post("/occupation/categorie/addComment",{id:categoryId.value,commentIds:commentId.value}).then(res=>{
    console.log(res)
    if(res.code!=0) return false;
    ElMessage.success("添加成功")
  })
}

defineExpose({
  init
});
</script>
