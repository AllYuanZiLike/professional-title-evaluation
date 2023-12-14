<template>
  <el-dialog v-model="visible" title="添加职评活动" :close-on-click-modal="false" :close-on-press-escape="false" width="30%">
    <el-select v-model="commentId" placeholder="请选择职评活动" multiple style="width: 100%;" @change="getCommentList(dataForm.categoryId)">
      <el-option v-for="item in dataForm.commentList" :key="item.id" :label="item.name" :value="item.id" />
    </el-select>
    <template v-slot:footer>
      <el-button @click="visible = false">{{ $t("cancel") }}</el-button>
      <el-button type="primary" @click="submitCommentId()">{{ $t("confirm") }}</el-button>
    </template>
  </el-dialog>
</template>

<script lang="ts" setup>
import {reactive, ref} from "vue";
import { ElMessage } from "element-plus";
import baseService from "@/service/baseService";

const visible = ref(false);

const commentId = ref([]);

const dataForm = reactive({
  categoryId:"",
  commentList:[]
})
const init = (id:string) => {
  dataForm.commentList = [];
  console.log(dataForm.commentList)
  dataForm.categoryId = id
  getCommentList(id)
};

const submitCommentId = ()=>{
  console.log(commentId.value)
  baseService.post("/occupation/categorie/addComment",{id:dataForm.categoryId,commentIds:commentId.value}).then(res=>{
    console.log(res)
    if(res.code!=0) return false;
    ElMessage.success("添加成功")
    visible.value = false;
  })
}

const emit = defineEmits(["refreshDataList"]);
const getCommentList = (id:string)=>{
  baseService.get("/occupation/categorie/SelectComment",{id:id}).then(res=>{
    console.log(res)
    if(res.code != 0) return false;
    dataForm.commentList=res.data.list;
    console.log(dataForm.commentList)
    visible.value = true;
    emit("refreshDataList");
  })

}

defineExpose({
  init,getCommentList
});
</script>
