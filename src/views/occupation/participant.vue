<template>
  <div class="mod-occupation__participant">
    <el-form :inline="true" :model="state.dataForm" @keyup.enter="state.getDataList()">
<!--      <el-form-item>-->
<!--        <el-input v-model="state.dataForm.name" placeholder="参与人姓名" clearable></el-input>-->
<!--      </el-form-item>-->
      <el-form-item  style="width: 8vw;">
<!--        <el-input style="width: 160px;" v-model="state.dataForm.applicationType" placeholder="申报类型" clearable></el-input>-->
        <el-select style="width: 160px;" v-model="state.dataForm.applicationType" clearable placeholder="申报类型">
          <el-option v-for="item in applicationTypes" :key="item.value" :label="item.label" :value="item.value"/>
        </el-select>
      </el-form-item>
<!--      <el-form-item>-->
<!--        <el-input v-model="state.dataForm.unitId" placeholder="推荐单位" clearable></el-input>-->
<!--      </el-form-item>-->
      <el-form-item style="width: 8vw;">
<!--        <el-input style="width: 160px;" v-model="state.dataForm.sGroup" placeholder="学科组" clearable></el-input>-->
        <el-select style="width: 160px;" v-model="state.dataForm.sGroup" clearable placeholder="学科组">
          <el-option v-for="item in groups" :key="item.value" :label="item.label" :value="item.value"/>
        </el-select>
      </el-form-item>
<!--      <el-form-item>-->
<!--        <el-input v-model="state.dataForm.professionalId" placeholder="申报专业" clearable></el-input>-->
<!--      </el-form-item>-->
<!--      <el-form-item>-->
<!--        <el-input v-model="state.dataForm.positionId" placeholder="申报职称" clearable></el-input>-->
<!--      </el-form-item>-->
      <el-form-item style="width: 8vw;">
<!--        <el-input style="width: 160px;" v-model="state.dataForm.recommendType" placeholder="推荐类型" clearable></el-input>-->
        <el-select style="width: 160px;" v-model="state.dataForm.recommendType" clearable placeholder="推荐类型">
          <el-option v-for="item in recommendTypes" :key="item.value" :label="item.label" :value="item.value"/>
        </el-select>
      </el-form-item>
      <el-form-item style="width: 8vw;">
<!--        <el-input style="width: 160px;" v-model="state.dataForm.honoraryName" placeholder="荣誉名称" clearable></el-input>-->
        <el-select style="width: 160px;" v-model="state.dataForm.honoraryName" clearable placeholder="荣誉名称">
          <el-option v-for="item in honorTypes" :key="item.value" :label="item.label" :value="item.value"/>
        </el-select>
      </el-form-item>
      <el-form-item style="width: 8vw;">
        <!--        <el-input style="width: 160px;" v-model="state.dataForm.honoraryName" placeholder="荣誉名称" clearable></el-input>-->
        <el-select style="width: 160px;" v-model="state.dataForm.isPass" clearable placeholder="是否通过">
          <el-option v-for="item in passTypes" :key="item.value" :label="item.label" :value="item.value"/>
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button @click="state.getDataList()">{{ $t("query") }}</el-button>
      </el-form-item>
      <el-form-item>
        <el-button type="info" @click="state.exportHandle()">{{ $t("export") }}</el-button>
      </el-form-item>
      <el-form-item>
        <el-button v-if="state.hasPermission('occupation:participant:save')" type="primary" @click="addOrUpdateHandle()">{{ $t("add") }}</el-button>
      </el-form-item>
      <el-form-item>
        <el-button v-if="state.hasPermission('occupation:participant:delete')" type="danger" @click="state.deleteHandle()">{{ $t("deleteBatch") }}</el-button>
      </el-form-item>
    </el-form>
    <el-table v-loading="state.dataListLoading" :data="state.dataList" border @selection-change="state.dataListSelectionChangeHandle" style="width: 100%">
      <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
      <el-table-column prop="name" label="参与人姓名" header-align="center" align="center"></el-table-column>
      <el-table-column prop="sexName" label="性别" header-align="center" align="center"></el-table-column>
      <el-table-column prop="applicationName" label="申报类型" header-align="center" align="center"></el-table-column>
      <el-table-column prop="unitName" label="推荐单位" header-align="center" align="center"></el-table-column>
      <el-table-column prop="groupName" label="学科组" header-align="center" align="center"></el-table-column>
      <el-table-column prop="professionalName" label="申报专业" header-align="center" align="center"></el-table-column>
      <el-table-column prop="positionName" label="申报职称" header-align="center" align="center"></el-table-column>
      <el-table-column prop="recommendName" label="推荐类型" header-align="center" align="center"></el-table-column>
      <el-table-column prop="honorary" label="荣誉名称" header-align="center" align="center"></el-table-column>
      <el-table-column prop="fileName" label="文件" header-align="center" align="center">
        <template v-slot="scope">
          <div style="cursor: pointer;" @click="PreviewFile(scope.$index)">{{scope.row.fileName}}</div>
        </template>
      </el-table-column>
      <el-table-column :label="$t('handle')" fixed="right" header-align="center" align="center" width="150">
        <template v-slot="scope">
          <el-button v-if="state.hasPermission('occupation:participant:update')" type="primary" link @click="addOrUpdateHandle(scope.row.id)">{{ $t("update") }}</el-button>
          <el-button v-if="state.hasPermission('occupation:participant:delete')" type="primary" link @click="state.deleteHandle(scope.row.id)">{{ $t("delete") }}</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination :current-page="state.page" :page-sizes="[10, 20, 50, 100]" :page-size="state.limit" :total="state.total" layout="total, sizes, prev, pager, next, jumper" @size-change="state.pageSizeChangeHandle" @current-change="state.pageCurrentChangeHandle"> </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update :key="addKey" ref="addOrUpdateRef" @refreshDataList="state.getDataList"></add-or-update>
    <el-dialog v-model="preFileVisible" title="文件预览" width="85%" style="">
      <iframe style="width: 100%;height: 65vh" :src="previewUrl"></iframe>
      <template #footer>
        <el-button @click="preFileVisible = false">关闭</el-button>
      </template>
    </el-dialog>
  </div>
</template>

<script lang="ts" setup>
import useView from "@/hooks/useView";
import { nextTick, reactive, ref, toRefs, watch } from "vue";
import AddOrUpdate from "./participant-add-or-update.vue";
import Template from "@/views/devtools/template.vue";
import {ElMessage} from "element-plus";

const view = reactive({
  getDataListURL: "/occupation/participant/page",
  getDataListIsPage: true,
  exportURL: "/occupation/participant/export",
  deleteURL: "/occupation/participant",
  deleteIsBatch: true,
  dataForm: {
    name: "",
    sex:0,
    applicationType: "",
    unitId: "",
    sGroup: "",
    isPass:"",
    professionalId: "",
    positionId: "",
    recommendType: "",
    honoraryName: "",
  },
  dataList:[]
});

const state = reactive({ ...useView(view), ...toRefs(view) });
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
const passTypes = reactive([
  {
    value:0,
    label:'未通过'
  },
  {
    value:1,
    label:'通过'
  }
])

const addKey = ref(0);
const addOrUpdateRef = ref();
const addOrUpdateHandle = ( id?: number) => {
  addKey.value++;
  nextTick(() => {
    addOrUpdateRef.value.init( id);
  });
};

const preFileVisible = ref(false)
const previewUrl = ref();
const PreviewFile = (index:number)=>{
  console.log(index)

  previewUrl.value = state.dataList[index].reserve01;
  if(previewUrl.value !== "") preFileVisible.value=true
  else ElMessage.warning("暂未上传文件");
}
</script>
