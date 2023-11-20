<template>
  <div class="mod-occupation__categorie">
    <el-form :inline="true" :model="state.dataForm" @keyup.enter="state.getDataList()">
      <el-form-item>
        <el-input v-model="state.dataForm.name" placeholder="类别名称" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="state.getDataList()">{{ $t("query") }}</el-button>
      </el-form-item>
<!--      <el-form-item>-->
<!--        <el-button type="info" @click="state.exportHandle()">{{ $t("export") }}</el-button>-->
<!--      </el-form-item>-->
      <el-form-item>
        <el-button v-if="state.hasPermission('occupation:categorie:save')" type="primary" @click="addOrUpdateHandle()">{{ $t("add") }}</el-button>
      </el-form-item>
      <el-form-item>
        <el-button v-if="state.hasPermission('occupation:categorie:delete')" type="danger" @click="state.deleteHandle()">{{ $t("deleteBatch") }}</el-button>
      </el-form-item>
      <el-form-item>
        <el-button v-if="state.hasPermission('occupation:categorie:history')" type="info" @click="checkHistory">历史</el-button>
      </el-form-item>
    </el-form>
<!--    -->
    <el-table @expand-change="loadComments" v-loading="state.dataListLoading" :data="state.dataList" border @selection-change="state.dataListSelectionChangeHandle" style="width: 100%">
      <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
      <el-table-column type="expand">
        <template #default="props">
          <el-table v-loading="loading" :data="commentsPage" :row-class-name="commentTableClassName">
            <el-table-column header-align="center" align="center" label="职评活动" prop="name" />
            <el-table-column header-align="center" align="center" label="评审进度">
              <template v-slot="scope">
                <el-progress :percentage="Math.floor((scope.row.voted)/(scope.row.judgeNumber)*100)" />
              </template>
            </el-table-column>
            <el-table-column :label="$t('handle')" fixed="right" header-align="center" align="center" width="150">
              <template v-slot="scope">
<!--                <el-button v-if="state.hasPermission('occupation:categorie:confirmRes') && scope.row.status == 1" type="primary" link @click="confirmResHandle(props.row.id,scope.row.id)">确认结果</el-button>-->
                <el-button v-if="state.hasPermission('occupation:categorie:deleteComment') && scope.row.status == 0" type="primary" link @click="deleteCommentHandle(props.row.id,scope.row.id)">{{ $t("delete") }}</el-button>
                <el-button v-if="state.hasPermission('occupation:categorie:checkResult') && scope.row.status != 0" type="primary" link @click="checkResultHandle(props.row.id,scope.row.id,scope.row.isPass)">查看{{scope.row.status == 1 ? '过程':'结果'}}</el-button>
              </template>
            </el-table-column>
          </el-table>
        </template>
      </el-table-column>
      <el-table-column prop="name" label="类别名称" header-align="center" align="center"></el-table-column>
      <el-table-column prop="info" label="类别简介" header-align="center" align="center"></el-table-column>
<!--      <el-table-column prop="createDate" label="创建时间" header-align="center" align="center"></el-table-column>-->
      <el-table-column :label="$t('handle')" fixed="right" header-align="center" align="center" width="150">
        <template v-slot="scope">
          <el-button v-if="state.hasPermission('occupation:categorie:addComment') && scope.row.status != 2" type="primary" link @click="addComment(scope.row.id)">添加职评活动</el-button>
          <el-button v-if="state.hasPermission('occupation:categorie:startCategory') && scope.row.status !=2" type="primary" link @click="startCategory(scope.row.id)">{{scope.row.status=='0'?'开启':'结束'}}类别</el-button>
          <el-button v-if="state.hasPermission('occupation:categorie:update') && scope.row.status == 0" type="primary" link @click="addOrUpdateHandle(scope.row.id)">{{ $t("update") }}</el-button>
          <el-button v-if="state.hasPermission('occupation:categorie:delete') && scope.row.status == 0" type="primary" link @click="state.deleteHandle(scope.row.id)">{{ $t("delete") }}</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination :current-page="state.page" :page-sizes="[10, 20, 50, 100]" :page-size="state.limit" :total="state.total" layout="total, sizes, prev, pager, next, jumper" @size-change="state.pageSizeChangeHandle" @current-change="state.pageCurrentChangeHandle"> </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update :key="addKey" ref="addOrUpdateRef" @refreshDataList="state.getDataList"></add-or-update>
    <addCommentCategory ref="addCommentKey" @refreshDataList="state.getDataList"></addCommentCategory>
    <CheckResult ref="checkResultKey"></CheckResult>
    <History ref="historyRef"></History>
  </div>
</template>

<script lang="ts" setup>
import useView from "@/hooks/useView";
import { nextTick, reactive, ref, toRefs } from "vue";
import AddOrUpdate from "./categorie-add-or-update.vue";
import addCommentCategory from "./categorie-comment-add.vue"
import CheckResult from "./result.vue"
import History from './historyCategory.vue'
import baseService from "@/service/baseService";
import {ElMessage, ElMessageBox} from "element-plus";
import Template from "@/views/devtools/template.vue";

const view = reactive({
  getDataListURL: "/occupation/categorie/page",
  getDataListIsPage: true,
  exportURL: "/occupation/categorie/export",
  deleteURL: "/occupation/categorie",
  deleteIsBatch: true,
  dataForm: {
    name: "",
    status:"",//类别状态
  },
  dataList:[{commentsPage:[] as any}]
});

const state = reactive({ ...useView(view), ...toRefs(view) });

// interface Comment {
//   name:string
//   id:string
//   isOver:number
//   judgeNumber:number
//   participantNum:number
//   status:number
//   voted:number
// }
interface Categoty {
  commentIds: Array<any>
  commentsPage: Array<any>
}

const loading = ref(false)
const commentsPage = ref([]);
const loadComments = (row: Categoty, expandedRows:any,isExpanded:boolean) => {
  console.log(row)
  console.log(expandedRows)
  console.log(isExpanded)
  baseService.get("/occupation/categorie/CommentsPage",{id:row.id}).then(res=>{
    console.log(res)
    if(res.code!=0) return false;
    commentsPage.value = res.data.list;
  })

}


const addKey = ref(0);
const addOrUpdateRef = ref();
const addOrUpdateHandle = (id?: number) => {
  addKey.value++;
  nextTick(() => {
    addOrUpdateRef.value.init(id);
  });
};

const addCommentKey = ref();
const addComment = (id:string)=>{
  nextTick(() => {
    addCommentKey.value.init(id);
  });
}


const checkResultKey = ref();
const checkResultHandle = (id:string,commentId:string)=>{
  nextTick(() => {
    checkResultKey.value.init(id,commentId);
  });
}

const deleteCommentHandle = (id:string,commentId:string) => {
  console.log(id,commentId)
  ElMessageBox.confirm("确定进行删除操作？","提示", {
    confirmButtonText:"确定",
    cancelButtonText: "取消",
    type: "warning"
  }).then(() => {
    baseService.get("/occupation/categorie/delComment", {id:id,commentId:commentId}).then(res=>{
      console.log(res)
      if(res.code!=0) return false;
      ElMessage.success("删除成功")
    })
  })

}

const historyRef = ref();
const checkHistory = ()=>{
  nextTick(() => {
    historyRef.value.init();
  });
}

const startCategory = (id:string)=>{
  console.log(id)
  baseService.put("/occupation/categorie/change", {id:id}).then(res=>{
    console.log(res)
    if(res.code != 0) return false
    ElMessage.success("启动成功")
    state.getDataList()
  })
}

const confirmResHandle = (categoryId:string,commentId:string)=>{
  console.log(categoryId,commentId)
}

const commentTableClassName = ({row, rowIndex,}: { row: Comment,rowIndex: number })=>{
  if(row.status == 2) return 'overRow';
}
</script>
<style lang="less">
.el-table .overRow{
  --el-table-tr-bg-color: #dbecfc;
}
</style>
