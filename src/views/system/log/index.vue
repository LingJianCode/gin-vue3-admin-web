<template>
  <div class="app-container">
    <div class="search-bar">
      <el-form ref="queryFormRef" :model="queryParams" :inline="true">
        <el-form-item prop="userId" label="userId">
          <el-input
            v-model="queryParams.userId"
            placeholder="userId"
            clearable
            @keyup.enter="handleQuery"
          />
        </el-form-item>
        <el-form-item prop="createTime" label="操作时间">
          <el-date-picker
            v-model="queryParams.createTime"
            :editable="false"
            class="!w-[240px]"
            type="daterange"
            range-separator="~"
            start-placeholder="开始时间"
            end-placeholder="截止时间"
            value-format="YYYY-MM-DD"
          />
        </el-form-item>

        <el-form-item>
          <el-button type="primary" icon="search" @click="handleQuery">搜索</el-button>
          <el-button icon="refresh" @click="handleResetQuery">重置</el-button>
        </el-form-item>
      </el-form>
    </div>

    <el-card shadow="never">
      <el-table v-loading="loading" :data="pageData" highlight-current-row border>
        <el-table-column label="操作人" prop="user.Nickname" />
        <el-table-column label="方法" prop="method" />
        <el-table-column label="URI" prop="path" />
        <el-table-column label="状态" prop="status" />
        <el-table-column label="IP" prop="ip" />
        <el-table-column align="left" label="请求" prop="path" width="80">
          <template #default="scope">
            <div>
              <el-popover v-if="scope.row.body" placement="left-start" :width="444">
                <div class="popover-box">
                  <pre>{{ fmtBody(scope.row.body) }}</pre>
                </div>
                <template #reference>
                  <el-icon style="cursor: pointer"><warning /></el-icon>
                </template>
              </el-popover>

              <span v-else>无</span>
            </div>
          </template>
        </el-table-column>
        <el-table-column align="left" label="响应" prop="path" width="80">
          <template #default="scope">
            <div>
              <el-popover v-if="scope.row.resp" placement="left-start" :width="444">
                <div class="popover-box">
                  <pre>{{ fmtBody(scope.row.resp) }}</pre>
                </div>
                <template #reference>
                  <el-icon style="cursor: pointer"><warning /></el-icon>
                </template>
              </el-popover>
              <span v-else>无</span>
            </div>
          </template>
        </el-table-column>
        <el-table-column label="浏览器" prop="agent" />
        <el-table-column label="执行时间(ms)" prop="latency" />
        <el-table-column label="操作时间" prop="createdAt" />
      </el-table>

      <pagination
        v-if="total > 0"
        v-model:total="total"
        v-model:page="queryParams.pageNum"
        v-model:limit="queryParams.pageSize"
        @pagination="handleQuery"
      />
    </el-card>
  </div>
</template>

<script setup>
defineOptions({
  name: "Log",
  inheritAttrs: false,
});

import LogAPI from "@/api/system/log.api";

const queryFormRef = ref();

const loading = ref(false);
const total = ref(0);

const queryParams = reactive({
  pageNum: 1,
  pageSize: 10,
  userId: 0,
  createTime: ["", ""],
});

// 日志表格数据
const pageData = ref();

/** 查询 */
function handleQuery() {
  loading.value = true;
  LogAPI.getPage(queryParams)
    .then((data) => {
      pageData.value = data.list;
      total.value = data.total;
    })
    .finally(() => {
      loading.value = false;
    });
}
/** 重置查询 */
function handleResetQuery() {
  queryFormRef.value.resetFields();
  queryParams.pageNum = 1;
  queryParams.createTime = undefined;
  handleQuery();
}

onMounted(() => {
  handleQuery();
});

const fmtBody = (value) => {
  try {
    return JSON.parse(value);
  } catch (_) {
    return value;
  }
};
</script>
