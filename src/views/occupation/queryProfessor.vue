<template>
  <el-dialog v-model="queryProVisible" title="选取专家" width="85%">
    <el-form inline :model="selectPro" :rules="rules" ref="selectProRef" @keyup.enter="getProJudgeList()">
      <el-form-item style="width: 14.7vw" label="选取人数">
        <el-input type="number" min="0" clearable v-model="selectPro.num" placeholder="请填写选取人数"></el-input>
      </el-form-item>
      <el-form-item style="width: 14vw">
        <el-input v-model="selectPro.unitName" clearable placeholder="请填写单位"></el-input>
      </el-form-item>
      <el-form-item style="width: 14vw">
        <el-select v-model="selectPro.ranks" clearable placeholder="请选择专业技术级别">
          <el-option value="高级"/>
          <el-option value="中级"/>
          <el-option value="低级"/>
        </el-select>
      </el-form-item>
      <el-form-item style="width:14vw">
        <el-input v-model="selectPro.subjects" clearable placeholder="请填写专业学科"></el-input>
      </el-form-item>
      <el-form-item style="width: 14vw">
<!--        <el-input v-model="selectPro.officeDate" placeholder="请选择" clearable></el-input>-->
        <el-date-picker v-model="selectPro.officeDate" clearable type="date" placeholder="请选择聘任年限" format="YYYY-MM-DD"/>
      </el-form-item>
      <el-form-item>
        <el-button @click="getProJudgeList" type="primary">获取</el-button>
      </el-form-item>
    </el-form>
    <el-table v-loading="state.dataListLoading" :data="proJudgeList" border @selection-change="state.dataListSelectionChangeHandle" style="width: 100%">
      <el-table-column prop="name" label="专家名称" header-align="center" align="center" width="70"></el-table-column>
      <el-table-column prop="staffId" label="职工编号" header-align="center" align="center" width="95"></el-table-column>
      <el-table-column prop="sex" label="性别" header-align="center" align="center" width="40">
        <template v-slot="scope">
          {{ state.getDictLabel("gender", scope.row.sex) }}
        </template>
      </el-table-column>
      <el-table-column prop="birth" label="出生日期" header-align="center" align="center" width="97"></el-table-column>
      <el-table-column prop="age" label="年龄" header-align="center" align="center" width="41"></el-table-column>
      <el-table-column prop="education" label="学历" header-align="center" align="center"></el-table-column>
      <el-table-column prop="degree" label="学位" header-align="center" align="center"></el-table-column>
      <el-table-column prop="subjects" label="学科" header-align="center" align="center"></el-table-column>
      <el-table-column prop="unitName" label="单位" header-align="center" align="center" show-overflow-tooltip></el-table-column>
      <el-table-column prop="occupation" label="职称" header-align="center" align="center"></el-table-column>
      <el-table-column prop="ranks" label="专业技术级别" header-align="center" align="center"></el-table-column>
      <el-table-column prop="officeDate" label="聘任起始日期" header-align="center" align="center" width="97"></el-table-column>
      <el-table-column :label="$t('handle')" fixed="right" header-align="center" align="center" width="150">
        <template v-slot="scope">
          <el-button v-if="state.hasPermission('occupation:professor:delete')" type="primary" link @click="state.deleteHandle(scope.row.id)">{{ $t("delete") }}</el-button>
        </template>
      </el-table-column>
    </el-table>
<!--    <el-pagination :current-page="state.page" :page-sizes="[10, 20, 50, 100]" :page-size="state.limit" :total="state.total" layout="total, sizes, prev, pager, next, jumper" @size-change="state.pageSizeChangeHandle" @current-change="state.pageCurrentChangeHandle"> </el-pagination>-->
    <template #footer>
      <span class="dialog-footer">
        <el-button @click="queryProVisible = false">取消</el-button>
        <el-button type="primary" @click="submitResList">提交</el-button>
      </span>
    </template>
  </el-dialog>
</template>

<script lang="ts" setup >
import {ref, reactive, toRefs} from 'vue'
import { ElMessage } from 'element-plus'
import baseService from "@/service/baseService";
import useView from "@/hooks/useView";

const emit = defineEmits(["refreshDataList"]);
const queryProVisible = ref(false);

const view = reactive({
  // getDataListURL: "/occupation/professor/page",
  getDataListIsPage: true,
  // exportURL: "/occupation/professor/export",
  // deleteURL: "/occupation/professor",
  // deleteIsBatch: true,
  procommentId:"",
  dataForm: {
    name:"",
    staffId:"",
    sex:"",
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
  }
});

const state = reactive({ ...useView(view), ...toRefs(view) });


const selectProRef = ref()
const selectPro = reactive({
  num:"1",
  unitName:"",
  ranks:"",
  subjects:"",
  officeDate:"",
  proCommentId:""
})

const rules = ref({
  num: [{ required: true, message: "有必填选项未填", trigger: "blur" }]
});

const init = (id:string) => {

  queryProVisible.value = true
  if (selectProRef.value) {
    selectProRef.value.resetFields();
  }
  selectPro.proCommentId = id
};

const proJudgeList = ref()
const getProJudgeList = ()=> {
  selectProRef.value.validate((valid: boolean) => {
    if (!valid) {
      return false;
    }
    baseService.get("/occupation/professor/find",selectPro).then(res=>{
      console.log(res)
      if(res.code != 0) return false
      proJudgeList.value = res.data.list;
      state.total = res.data.total;
    })
  })
}

const submitResList = () => {
  let proJudgeIds = proJudgeList.value.map((item:any)=>{
    return item.id
  })

  console.log(proJudgeIds)
  baseService.post("/occupation/commentprofessor/save", {commentId: selectPro.proCommentId, proJudgeIds: proJudgeIds}).then((res) => {
    ElMessage.success({
      message: "提交成功",
      duration: 500,
      onClose: () => {
        queryProVisible.value = false;
        emit("refreshDataList");
      }
    });
  })
}

defineExpose({
  init
});
</script>
