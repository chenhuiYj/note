# Vue3 + Element Plus 表格跨分页多选记忆选中方案

## 一、业务场景描述
后台管理系统表格批量操作常见问题：
1. 切换页码后，上一页选中状态丢失
2. 全选会影响所有页面，不符合仅操作当前页需求
3. 批量删除/导出时跨页选中，导致误操作

需求：
- 每页选中状态独立缓存
- 切回页码自动回显之前勾选
- 批量操作仅提交当前页选中数据

## 二、实现思路
1. 表格设置 row-key="id" 保证行唯一
2. 使用 pageSelectedMap 按页码缓存选中 ID
3. 切换页码时自动同步选中状态
4. 批量操作只取当前页选中，不跨页合并

## 三、完整代码
```vue
<template>
  <div class="table-page">
    <div style="margin-bottom: 16px">
      <el-button type="danger" @click="handleBatchDelete" :disabled="!currentSelection.length">
        批量删除（当前页）
      </el-button>
    </div>

    <el-table
      ref="tableRef"
      v-loading="loading"
      :data="tableList"
      row-key="id"
      @selection-change="handleSelectionChange"
      border
      style="width: 100%; margin-bottom: 20px"
    >
      <el-table-column type="selection" width="55" />
      <el-table-column label="ID" prop="id" />
      <el-table-column label="名称" prop="name" />
    </el-table>

    <el-pagination
      v-model:current-page="pageNo"
      v-model:page-size="pageSize"
      :total="100"
      layout="total, sizes, prev, pager, next, jumper"
      @size-change="handlePageChange"
      @current-change="handlePageChange"
    />
  </div>
</template>

<script setup>
import { ref, reactive, onMounted } from 'vue'
import { ElMessage, ElMessageBox } from 'element-plus'

const tableRef = ref(null)
const pageNo = ref(1)
const pageSize = ref(10)
const tableList = ref([])
const loading = ref(false)

const currentSelection = ref([])
const pageSelectedMap = reactive({})

const getTableList = async () => {
  loading.value = true
  await new Promise(resolve => setTimeout(resolve, 200))
  const list = []
  const start = (pageNo.value - 1) * pageSize.value
  for (let i = 0; i < pageSize.value; i++) {
    list.push({ id: start + i + 1, name: `数据${start + i + 1}` })
  }
  tableList.value = list
  syncSelectedStatus()
  loading.value = false
}

const handlePageChange = () => {
  currentSelection.value = []
  getTableList()
}

const handleSelectionChange = (val) => {
  currentSelection.value = val
  pageSelectedMap[pageNo.value] = val.map(item => item.id)
}

const syncSelectedStatus = () => {
  tableRef.value?.clearSelection()
  const ids = pageSelectedMap[pageNo.value] || []
  tableList.value.forEach(row => {
    if (ids.includes(row.id)) {
      tableRef.value?.toggleRowSelection(row, true)
    }
  })
}

const handleBatchDelete = async () => {
  if (!currentSelection.value.length) {
    ElMessage.warning('请选择数据')
    return
  }
  const ids = currentSelection.value.map(item => item.id)
  await ElMessageBox.confirm('确定删除当前页选中数据？')
  ElMessage.success(`删除成功：${ids.join(',')}`)
  getTableList()
}

onMounted(() => {
  getTableList()
})
</script>

<style scoped>
.table-page {
  padding: 20px;
  background: #fff;
}
</style>
```
## 四、功能效果
第 1 页选中 1、2、3 → 缓存 {1: [1,2,3]}

第 2 页选中 11、12、13 → 缓存 {2: [11,12,13]}

批量删除仅提交当前页选中

切换页码自动回显对应页选中状态

多页选中互不干扰

## 五、面试回答话术
问：表格多选切换页码后选中状态丢失怎么解决？

答：通过按页码缓存选中 ID 实现：

表格设置 row-key="id" 保证行唯一；

用对象按页码存储每页选中 ID 数组；

切换页码时清空选中，并根据缓存 ID 手动回显；

批量操作只取当前页选中，避免跨页干扰。