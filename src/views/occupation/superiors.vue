<template>
  <div class="mod-occupation__comment">
    <el-drawer v-model="drawerSuper"  title="被评审人列表" direction="rtl" size="85%">
      <el-form :inline="true" :model="state.dataForm" @keyup.enter="getParticipants">
        <el-form-item style="width: 8vw;">
          <el-select v-model="state.dataForm.honoraryName" clearable filterable placeholder="职称类型">
            <el-option v-for="item in positions" :key="item.value" :label="item.label" :value="item.value"/>
          </el-select>
        </el-form-item>
        <el-form-item style="width: 8vw;">
          <!--        <el-input v-model="state.dataForm.sgroup" placeholder="学科组" clearable></el-input>-->
          <el-select v-model="state.dataForm.sgroup" clearable filterable placeholder="学科组">
            <el-option v-for="item in groups" :key="item.value" :label="item.label" :value="item.value"/>
          </el-select>
        </el-form-item>
        <el-form-item style="width: 8vw;">
          <!--        <el-input v-model="state.dataForm.recommendType" placeholder="推荐类型" clearable></el-input>-->
          <el-select v-model="state.dataForm.recommendType" clearable filterable placeholder="推荐类型">
            <el-option v-for="item in recommendTypes" :key="item.value" :label="item.label" :value="item.value"/>
          </el-select>
        </el-form-item>
        <el-form-item style="width: 8vw;">
          <!--        <el-input v-model="state.dataForm.applicationType" placeholder="申报类型" clearable></el-input>-->
          <el-select v-model="state.dataForm.applicationType" clearable filterable placeholder="申报类型">
            <el-option v-for="item in applicationTypes" :key="item.value" :label="item.label" :value="item.value"/>
          </el-select>
        </el-form-item>
        <el-form-item>
          <el-button @click="getParticipants()">{{ $t("query") }}</el-button>
        </el-form-item>
        <el-form-item>
          <el-button v-if="state.hasPermission('occupation:comment:save')" type="primary" @click="superiorAddHandle(state.commentId)">{{ $t("add") }}</el-button>
        </el-form-item>
        <el-form-item>
          <el-button v-if="state.hasPermission('occupation:comment:delete')" type="danger" @click="state.deleteHandle()">{{ $t("deleteBatch") }}</el-button>
        </el-form-item>
      </el-form>
      <el-table v-loading="state.dataListLoading" :data="participants" border @selection-change="state.dataListSelectionChangeHandle" style="width: 100%">
        <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
        <el-table-column prop="name" :label="$t('user.username')" sortable="custom" header-align="center" align="center"></el-table-column>
        <el-table-column prop="honoraryName" label="职称类型" header-align="center" align="center"></el-table-column>
        <el-table-column prop="applicationName" label="申报类型" header-align="center" align="center"></el-table-column>
        <el-table-column prop="groupName" label="学科组" header-align="center" align="center"></el-table-column>
        <el-table-column prop="recommendName" label="推荐类型" header-align="center" align="center"></el-table-column>
        <el-table-column :label="$t('handle')" fixed="right" header-align="center" align="center" width="150">
          <template v-slot="scope">
            <el-button v-if="state.hasPermission('occupation:comment:delete')" type="primary" link @click="state.deleteHandle(scope.row.id)">{{ $t("delete") }}</el-button>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination :current-page="state.page" :page-sizes="[10, 20, 50, 100]" :page-size="state.limit" :total="state.total" layout="total, sizes, prev, pager, next, jumper" @size-change="state.pageSizeChangeHandle" @current-change="state.pageCurrentChangeHandle"> </el-pagination>
      <!-- 弹窗, 新增  -->
      <SuperiorAdd ref="superiorAddRef" @getSuperiorsList="getParticipants()"></SuperiorAdd>
    </el-drawer>
  </div>
</template>

<script lang="ts" setup>
import useView from "@/hooks/useView";
import { nextTick, reactive, ref, toRefs, watch } from "vue";
import SuperiorAdd from "./superiors-add.vue";
import baseService from "@/service/baseService";

const view = reactive({
  commentId:"",
  page:1,
  limit:10,
  dataForm: {
    commentId:"",
    superiors: "",
    superiorsName:"",
    applicationType: "",
    sgroup: "",
    positionId: "",
    recommendType: "",
    honoraryName: "",
  },
});

const state = reactive({ ...useView(view), ...toRefs(view) });
const drawerSuper = ref(false);
const participants = ref([]);
const statusText = reactive([
  {
    value:0,
    label:'未开始'
  },
  {
    value:1,
    label:'进行中'
  },
  {
    value:2,
    label:'已结束'
  },
])
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
const positions = reactive([
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

const init = (id: string) => {
  drawerSuper.value = true;
  state.commentId = id;
  getParticipants()
};

const getParticipants = ()=>{
  baseService.get("/occupation/participant/getParticipants",{page: state.page, limit:  state.limit,commentId:state.commentId,...{applicationType:state.dataForm.applicationType,sgroup:state.dataForm.sgroup,recommendType:state.dataForm.recommendType,honoraryName:state.dataForm.honoraryName}}).then(res=>{
    console.log(res)
    if(res.code != 0) return false
    participants.value = res.data.list;
  })
}
const superiorAddRef = ref();
const superiorAddHandle = (id: string) => {
  nextTick(() => {
    superiorAddRef.value.init(id);
  });
};

defineExpose({
  init
})
</script>
