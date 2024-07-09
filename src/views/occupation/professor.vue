<template>
  <div class="mod-occupation__professor">
    <el-form :inline="true" :model="state.dataForm" @keyup.enter="state.getDataList()">
      <el-form-item style="width: 11vw">
        <el-input v-model="state.dataForm.name" placeholder="专家名称" clearable></el-input>
      </el-form-item>
      <el-form-item style="width: 11vw">
        <el-input v-model="state.dataForm.unitName" placeholder="单位" clearable></el-input>
      </el-form-item>
<!--      <el-form-item>-->
<!--            <ren-radio-group v-model="state.dataForm.sex" dict-type="gender"></ren-radio-group>-->
<!--      </el-form-item>-->
      <el-form-item style="width: 11vw">
        <el-input v-model="state.dataForm.occupation" placeholder="职称" clearable></el-input>
      </el-form-item>
<!--      <el-form-item>-->
<!--            <el-radio-group v-model="state.dataForm.isOffice">-->
<!--              <el-radio :label="0">单选</el-radio>-->
<!--            </el-radio-group>-->
<!--      </el-form-item>-->
      <el-form-item>
        <el-button type="warning" @click="importHandle()">导入</el-button>
      </el-form-item>
      <el-form-item>
        <el-button @click="state.getDataList()">{{ $t("query") }}</el-button>
      </el-form-item>
      <el-form-item>
        <el-button type="info" @click="state.exportHandle()">{{ $t("export") }}</el-button>
      </el-form-item>
      <el-form-item>
        <el-button v-if="state.hasPermission('occupation:professor:save')" type="primary" @click="addOrUpdateHandle()">{{ $t("add") }}</el-button>
      </el-form-item>
<!--      <el-form-item>-->
<!--        <el-button v-if="state.hasPermission('occupation:professor:delete')" type="danger" @click="state.deleteHandle()">{{ $t("deleteBatch") }}</el-button>-->
<!--      </el-form-item>-->
    </el-form>
    <el-table v-loading="state.dataListLoading" :data="state.dataList" border @selection-change="state.dataListSelectionChangeHandle" style="width: 100%">
<!--      <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>-->
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
          <el-button v-if="state.hasPermission('occupation:professor:update')" type="primary" link @click="addOrUpdateHandle(scope.row.id)">{{ $t("update") }}</el-button>
          <el-button v-if="state.hasPermission('occupation:professor:delete')" type="primary" link @click="state.deleteHandle(scope.row.id)">{{ $t("delete") }}</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination :current-page="state.page" :page-sizes="[10, 20, 50, 100]" :page-size="state.limit" :total="state.total" layout="total, sizes, prev, pager, next, jumper" @size-change="state.pageSizeChangeHandle" @current-change="state.pageCurrentChangeHandle"> </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <ImportProfess ref="importRef" @refreshDataList="state.getDataList"></ImportProfess>
    <add-or-update :key="addKey" ref="addOrUpdateRef" @refreshDataList="state.getDataList"></add-or-update>
  </div>
</template>

<script lang="ts" setup>
import useView from "@/hooks/useView";
import { nextTick, reactive, ref, toRefs, watch } from "vue";
import ImportProfess from "./profess-import.vue";
import AddOrUpdate from "./professor-add-or-update.vue";

const view = reactive({
  getDataListURL: "/occupation/professor/page",
  getDataListIsPage: true,
  exportURL: "/occupation/professor/export",
  deleteURL: "/occupation/professor",
  deleteIsBatch: true,
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

const importRef = ref();
const importHandle = () => {
  importRef.value.init();
};

const addKey = ref(0);
const addOrUpdateRef = ref();
const addOrUpdateHandle = (id?: number) => {
  addKey.value++;
  nextTick(() => {
    addOrUpdateRef.value.init(id);
  });
};
</script>
