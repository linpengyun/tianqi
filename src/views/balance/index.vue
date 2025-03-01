<template>
  <div class="balance-container">
    <el-card>
      <template #header>
        <div class="card-header">
          <div class="header-left">
            <el-input
              v-model="searchQuery"
              placeholder="请输入关键字搜索"
              style="width: 200px"
              clearable
            >
              <template #prefix>
                <el-icon><Search /></el-icon>
              </template>
            </el-input>
          </div>
          <div class="header-right">
            <el-button type="primary" @click="handleAdd">
              <el-icon><Plus /></el-icon>新增分账规则
            </el-button>
          </div>
        </div>
      </template>

      <el-table :data="balanceList" style="width: 100%">
        <el-table-column prop="id" label="ID" width="80" />
        <el-table-column prop="projectName" label="项目名称" />
        <el-table-column prop="accountType" label="账户类型">
          <template #default="{ row }">
            <el-tag>{{ row.accountType === 1 ? '对公账户' : '个人账户' }}</el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="accountName" label="账户名称" />
        <el-table-column prop="accountNo" label="账号" />
        <el-table-column prop="bankName" label="开户行" />
        <el-table-column prop="ratio" label="分账比例">
          <template #default="{ row }">
            {{ row.ratio }}%
          </template>
        </el-table-column>
        <el-table-column prop="status" label="状态">
          <template #default="{ row }">
            <el-tag :type="row.status === 1 ? 'success' : 'danger'">
              {{ row.status === 1 ? '启用' : '禁用' }}
            </el-tag>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="200">
          <template #default="{ row }">
            <el-button type="primary" link @click="handleEdit(row)">
              <el-icon><Edit /></el-icon>编辑
            </el-button>
            <el-button type="danger" link @click="handleDelete(row)">
              <el-icon><Delete /></el-icon>删除
            </el-button>
          </template>
        </el-table-column>
      </el-table>

      <div class="pagination-container">
        <el-pagination
          v-model:current-page="currentPage"
          v-model:page-size="pageSize"
          :total="total"
          :page-sizes="[10, 20, 50, 100]"
          layout="total, sizes, prev, pager, next, jumper"
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
        />
      </div>
    </el-card>

    <!-- 分账规则表单对话框 -->
    <el-dialog
      :title="dialogTitle"
      v-model="dialogVisible"
      width="500px"
      @close="handleDialogClose"
    >
      <el-form
        ref="formRef"
        :model="form"
        :rules="rules"
        label-width="100px"
      >
        <el-form-item label="项目名称" prop="projectName">
          <el-input v-model="form.projectName" placeholder="请输入项目名称" />
        </el-form-item>
        <el-form-item label="账户类型" prop="accountType">
          <el-select v-model="form.accountType" placeholder="请选择账户类型">
            <el-option label="对公账户" :value="1" />
            <el-option label="个人账户" :value="2" />
          </el-select>
        </el-form-item>
        <el-form-item label="账户名称" prop="accountName">
          <el-input v-model="form.accountName" placeholder="请输入账户名称" />
        </el-form-item>
        <el-form-item label="账号" prop="accountNo">
          <el-input v-model="form.accountNo" placeholder="请输入账号" />
        </el-form-item>
        <el-form-item label="开户行" prop="bankName">
          <el-input v-model="form.bankName" placeholder="请输入开户行" />
        </el-form-item>
        <el-form-item label="分账比例" prop="ratio">
          <el-input-number
            v-model="form.ratio"
            :min="0"
            :max="100"
            :precision="2"
            :step="0.01"
            style="width: 180px"
          />
          <span class="ratio-unit">%</span>
        </el-form-item>
        <el-form-item label="状态" prop="status">
          <el-switch
            v-model="form.status"
            :active-value="1"
            :inactive-value="0"
          />
        </el-form-item>
      </el-form>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="dialogVisible = false">取消</el-button>
          <el-button type="primary" @click="handleSubmit">确定</el-button>
        </span>
      </template>
    </el-dialog>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive } from 'vue'
import { ElMessage, ElMessageBox } from 'element-plus'
import { Search, Plus, Edit, Delete } from '@element-plus/icons-vue'
import type { FormInstance, FormRules } from 'element-plus'

// 搜索相关
const searchQuery = ref('')

// 分页相关
const currentPage = ref(1)
const pageSize = ref(10)
const total = ref(0)

// 表格数据
const balanceList = ref([
  {
    id: 1,
    projectName: '示例项目',
    accountType: 1,
    accountName: '示例公司',
    accountNo: '6222123456789012345',
    bankName: '示例银行',
    ratio: 30,
    status: 1
  }
])

// 表单相关
const dialogVisible = ref(false)
const dialogTitle = ref('')
const formRef = ref<FormInstance>()

const form = reactive({
  id: undefined,
  projectName: '',
  accountType: 1,
  accountName: '',
  accountNo: '',
  bankName: '',
  ratio: 0,
  status: 1
})

const rules: FormRules = {
  projectName: [{ required: true, message: '请输入项目名称', trigger: 'blur' }],
  accountType: [{ required: true, message: '请选择账户类型', trigger: 'change' }],
  accountName: [{ required: true, message: '请输入账户名称', trigger: 'blur' }],
  accountNo: [{ required: true, message: '请输入账号', trigger: 'blur' }],
  bankName: [{ required: true, message: '请输入开户行', trigger: 'blur' }],
  ratio: [{ required: true, message: '请输入分账比例', trigger: 'blur' }]
}

// 处理新增
const handleAdd = () => {
  dialogTitle.value = '新增分账规则'
  dialogVisible.value = true
  Object.assign(form, {
    id: undefined,
    projectName: '',
    accountType: 1,
    accountName: '',
    accountNo: '',
    bankName: '',
    ratio: 0,
    status: 1
  })
}

// 处理编辑
const handleEdit = (row: any) => {
  dialogTitle.value = '编辑分账规则'
  dialogVisible.value = true
  Object.assign(form, row)
}

// 处理删除
const handleDelete = (row: any) => {
  ElMessageBox.confirm('确认删除该分账规则吗？', '提示', {
    type: 'warning'
  }).then(async () => {
    try {
      // TODO: 调用删除接口
      ElMessage.success('删除成功')
    } catch (error) {
      console.error('删除失败:', error)
      ElMessage.error('删除失败')
    }
  })
}

// 处理提交
const handleSubmit = async () => {
  if (!formRef.value) return
  await formRef.value.validate(async (valid) => {
    if (valid) {
      try {
        // TODO: 调用保存接口
        ElMessage.success(form.id ? '更新成功' : '新增成功')
        dialogVisible.value = false
        // 刷新列表
        // getBalanceList()
      } catch (error) {
        console.error('保存失败:', error)
        ElMessage.error('保存失败')
      }
    }
  })
}

// 处理对话框关闭
const handleDialogClose = () => {
  if (formRef.value) {
    formRef.value.resetFields()
  }
}

// 处理分页大小变化
const handleSizeChange = (val: number) => {
  pageSize.value = val
  currentPage.value = 1
  // getBalanceList()
}

// 处理页码变化
const handleCurrentChange = (val: number) => {
  currentPage.value = val
  // getBalanceList()
}

// 初始化
// getBalanceList()
</script>

<style scoped>
.balance-container {
  padding: 20px;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.pagination-container {
  margin-top: 20px;
  display: flex;
  justify-content: flex-end;
}

.ratio-unit {
  margin-left: 8px;
}
</style>