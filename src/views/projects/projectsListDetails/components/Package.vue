<template>
  <a-table class="my-4" :columns="tableColumns" :dataSource="tableList" :pagination="pagination">
    <template #bodyCell="{ column, record, index }">
      <template v-if="column.dataIndex === 'action'">
        <label class="text-[#E2B578] cursor-pointer" v-if="record.domain === ''"
          @click="goDeploy(record.projectId ,record.workflowId, record.workflowDetailId)">Deploy</label>
        <label class="text-[#E2B578] cursor-pointer ml-2" v-else
          @click="goView(record.workflowId, record.workflowDetailId, record.id)">View</label>

      </template>
    </template>
  </a-table>
  <CustomMsg :showMsg="showMsg" :msgParam="msgParam"></CustomMsg>
</template>
<script setup lang="ts">
import { computed, onMounted, reactive, ref, toRefs } from 'vue';
import { formatDateToLocale } from '@/utils/dateUtil';
import { useRouter } from "vue-router";
import { apiGetProjectsPackages, apiProjectsDeploy } from "@/apis/projects";
import CustomMsg from '@/components/CustomMsg.vue';
import { message } from 'ant-design-vue';

const props = defineProps({
  detailId: String,
  pageType: String,
  packageListData: Array,
});
const { pageType, detailId, packageListData } = toRefs(props);

const router = useRouter();
const showMsg = ref(false);
const msgParam = ref({
  id: 0,
  workflowsId: 0,
  workflowDetailId: 0,
  projectType: 2,
});
const tableList = ref([]);
const pagination = ref();

const tableColumns = computed<any[]>(() => [
  {
    title: 'Package Name',
    dataIndex: 'name',
    align: 'center',
    ellipsis: 'fixed',
    key: 'name',
  },
  {
    title: 'Version',
    dataIndex: 'version',
    align: 'center',
    ellipsis: 'fixed',
    key: 'type',
  },
  {
    title: 'Branch',
    dataIndex: 'branch',
    key: 'branch',
    ellipsis: 'fixed',
    align: 'center',
  },
  {
    title: 'Build Time',
    dataIndex: 'buildTime',
    align: 'center',
    ellipsis: 'fixed',
    key: 'buildTime',
    customRender: ({ text: date }) => formatDateToLocale(date).format("YYYY/MM/DD HH:mm:ss"),
  },
  {
    title: 'Domains',
    dataIndex: 'domain',
    key: 'domain',
    ellipsis: 'fixed',
    align: 'center',
  },
  {
    title: 'Action',
    dataIndex: 'action',
    align: 'center',
    width: '150px',
  },
]);

const paginationVal = reactive({
  // ???????????????
  pageSize: 10, // ?????????????????????
  current: 1, // ?????????
  total: 10, // ??????
  size: 'small',
  position: ['bottomCenter'], //???????????????????????????
  hideOnSinglePage: false, // ????????????????????????????????????
  showQuickJumper: false, // ?????????????????????????????????
  showSizeChanger: false, // ?????????????????? pageSize
  pageSizeOptions: ['10', '20', '30'], // ?????????????????????????????????
  onShowSizeChange: (current: number, pagesize: number) => {
    // ?????? pageSize????????????
    pagination.current = current;
    pagination.pageSize = pagesize;
    getProjectsPackage();
  },
  onChange: (current: number) => {
    // ???????????????????????????
    pagination.current = current;
    getProjectsPackage();
  },
  // showTotal: total => `?????????${total}???`, // ??????????????????
});
onMounted(() => {
  if (pageType?.value === 'project') {
    pagination.value = paginationVal;
    getProjectsPackage()
  } else {
    tableList.value = packageListData.value;
    pagination.value = false;
  }
})

const getProjectsPackage = async () => {
  try {
    const params = {
      page: pagination.current,
      size: pagination.pageSize,
    }
    const { data } = await apiGetProjectsPackages(detailId?.value, params);
    tableList.value = data.data;
    pagination.total = data.total

  } catch (error: any) {
    console.log("erro:", error)
  } finally {
    // loading.value = false;
  }
}

const goDeploy = async (projectId:number, workflowId: number, workflowDetailId: number) => {

  try {
    const params = ref({
      id: projectId,
      workflowsId: workflowId,
      workflowDetailId: workflowDetailId,
    });
    const { data } = await apiProjectsDeploy(params.value); 
    showMsg.value = true;
    msgParam.value.id = projectId;
    msgParam.value.workflowsId = data.workflowId;
    msgParam.value.workflowDetailId = data.detailId;
    setTimeout(function () {
      showMsg.value = false;
    }, 3000)
  } catch (error: any) {
    console.log("erro:", error)
    message.error(error.response.data.message);
  }
}

const goView = (workflowId: number, workflowDetailId: number, packageId:number) => {
  router.push("/projects/" + workflowId + "/frontend-details/" + workflowDetailId + "/" + packageId);
}

// const downloadAbi = (val: any) => {
//   console.log(val, 'val')
// }

defineExpose({
  getProjectsPackage
})
</script>