<template>
  <div class="mod-occupation__comment">
    <el-drawer v-model="drawerSuper"  title="被评审人列表" direction="rtl" size="85%">
      <el-form :inline="true" :model="state.dataForm" @keyup.enter="state.getDataList()">
        <el-form-item>
          <el-select v-model="state.dataForm.position" clearable filterable placeholder="请选择职称">
            <el-option v-for="item in positions" :key="item.value" :label="item.label" :value="item.value"/>
          </el-select>
        </el-form-item>
        <el-form-item>
          <!--        <el-input v-model="state.dataForm.sGroup" placeholder="学科组" clearable></el-input>-->
          <el-select v-model="state.dataForm.sGroup" clearable filterable placeholder="请选择学科">
            <el-option v-for="item in groups" :key="item.value" :label="item.label" :value="item.value"/>
          </el-select>
        </el-form-item>
        <el-form-item>
          <!--        <el-input v-model="state.dataForm.recommendType" placeholder="推荐类型" clearable></el-input>-->
          <el-select v-model="state.dataForm.recommendType" clearable filterable placeholder="请选择推荐类型">
            <el-option v-for="item in recommendTypes" :key="item.value" :label="item.label" :value="item.value"/>
          </el-select>
        </el-form-item>
        <el-form-item>
          <!--        <el-input v-model="state.dataForm.applicationType" placeholder="申报类型" clearable></el-input>-->
          <el-select v-model="state.dataForm.applicationType" clearable filterable placeholder="请选择申报类型">
            <el-option v-for="item in applicationTypes" :key="item.value" :label="item.label" :value="item.value"/>
          </el-select>
        </el-form-item>
        <el-form-item>
          <el-button @click="state.getDataList()">{{ $t("query") }}</el-button>
        </el-form-item>
        <el-form-item>
          <el-button v-if="state.hasPermission('occupation:comment:save')" type="primary" @click="superiorAddHandle(state.commentId)">{{ $t("add") }}</el-button>
        </el-form-item>
        <el-form-item>
          <el-button v-if="state.hasPermission('occupation:comment:delete')" type="danger" @click="state.deleteHandle()">{{ $t("deleteBatch") }}</el-button>
        </el-form-item>
      </el-form>
      <el-table v-loading="state.dataListLoading" :data="dataList" border @selection-change="state.dataListSelectionChangeHandle" style="width: 100%">
        <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
        <el-table-column prop="name" label="评议活动名称" header-align="center" align="center"></el-table-column>
        <el-table-column prop="info" label="评议活动介绍" header-align="center" align="center">
          <template v-slot="scope">
            <p v-html="scope.row.info"></p>
          </template>
        </el-table-column>
        <!--      <el-table-column prop="superiors" label="上级评审ids" header-align="center" align="center"></el-table-column>-->
        <el-table-column prop="indicator" label="指标数" header-align="center" align="center"></el-table-column>
        <el-table-column prop="positionName" label="评审职称等级" header-align="center" align="center"></el-table-column>
        <el-table-column prop="groupName" label="学科组" header-align="center" align="center"></el-table-column>
        <el-table-column prop="recommendName" label="推荐类型" header-align="center" align="center"></el-table-column>
        <el-table-column prop="applicationName" label="申报类型" header-align="center" align="center"></el-table-column>
        <!--      <el-table-column prop="competitiveNum" label="竞争性参评人数（竞争型总人数）" header-align="center" align="center"></el-table-column>-->
        <!--      <el-table-column prop="competitiveIndicator" label="竞争性指标数" header-align="center" align="center"></el-table-column>-->
        <!--      <el-table-column prop="participantNum" label="选取人数（参与人数）" header-align="center" align="center"></el-table-column>-->
        <!--      <el-table-column prop="residualIndicator" label="剩余指标数" header-align="center" align="center"></el-table-column>-->
        <el-table-column prop="link" label="第几环节" header-align="center" align="center"></el-table-column>
        <el-table-column prop="stage" label="第几阶段" header-align="center" align="center"></el-table-column>
        <!--      <el-table-column prop="indicatorType" label="指标数类型（+1/等额）" header-align="center" align="center"></el-table-column>-->
        <el-table-column prop="statusName" label="活动状态" header-align="center" align="center"></el-table-column>
        <el-table-column :label="$t('handle')" fixed="right" header-align="center" align="center" width="150">
          <template v-slot="scope">
            <el-button v-if="state.hasPermission('occupation:comment:addSuperiors')" type="primary" link @click="addOrUpdateHandle(scope.row.id)">添加被评审人</el-button>
            <el-button v-if="state.hasPermission('occupation:comment:update')" type="primary" link @click="addOrUpdateHandle(scope.row.id)">{{ $t("update") }}</el-button>
            <el-button v-if="state.hasPermission('occupation:comment:delete')" type="primary" link @click="state.deleteHandle(scope.row.id)">{{ $t("delete") }}</el-button>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination :current-page="state.page" :page-sizes="[10, 20, 50, 100]" :page-size="state.limit" :total="state.total" layout="total, sizes, prev, pager, next, jumper" @size-change="state.pageSizeChangeHandle" @current-change="state.pageCurrentChangeHandle"> </el-pagination>
      <!-- 弹窗, 新增  -->
      <SuperiorAdd ref="superiorAddRef"></SuperiorAdd>
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
  limit:10
});

const state = reactive({ ...useView(view), ...toRefs(view) });
const drawerSuper = ref(false);
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
  baseService.get("/occupation/participant/getParticipants",{page: state.page, limit:  state.limit,commentId:state.commentId}).then(res=>{
    console.log(res)
  })

};

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
