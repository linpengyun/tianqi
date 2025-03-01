<template>
  <div class="menu-container">
    <div class="search-bar">
      <el-input
        v-model="searchQuery"
        placeholder="请输入菜单名称搜索"
        class="search-input"
        clearable
        @clear="handleSearch"
        @keyup.enter="handleSearch"
      >
        <template #append>
          <el-button @click="handleSearch">
            <el-icon><Search /></el-icon>
          </el-button>
        </template>
      </el-input>
      <el-button type="primary" @click="handleAdd">
        <el-icon><Plus /></el-icon>添加菜单
      </el-button>
    </div>

    <el-table :data="menuList" border style="width: 100%" row-key="id">
      <el-table-column prop="id" label="ID" width="80" />
      <el-table-column prop="name" label="菜单名称" width="150" />
      <el-table-column prop="path" label="路径" width="150" />
      <el-table-column prop="component" label="组件" width="180" />
      <el-table-column prop="icon" label="图标" width="100">
        <template #default="{ row }">
          <el-icon>
            <component :is="row.icon" />
          </el-icon>
        </template>
      </el-table-column>
      <el-table-column prop="sort" label="排序" width="80" />
      <el-table-column prop="status" label="状态" width="100">
        <template #default="{ row }">
          <el-tag :type="row.status === 1 ? 'success' : 'danger'">
            {{ row.status === 1 ? '启用' : '禁用' }}
          </el-tag>
        </template>
      </el-table-column>
      <el-table-column label="操作" width="200" fixed="right">
        <template #default="{ row }">
          <el-button-group>
            <el-button type="primary" @click="handleEdit(row)">
              <el-icon><Edit /></el-icon>编辑
            </el-button>
            <el-button type="danger" @click="handleDelete(row)">
              <el-icon><Delete /></el-icon>删除
            </el-button>
          </el-button-group>
        </template>
      </el-table-column>
    </el-table>

    <el-pagination
      v-model:current-page="currentPage"
      v-model:page-size="pageSize"
      :total="total"
      :page-sizes="[10, 20, 50, 100]"
      layout="total, sizes, prev, pager, next, jumper"
      class="pagination"
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
    />

    <el-dialog
      v-model="dialogVisible"
      :title="dialogType === 'add' ? '添加菜单' : '编辑菜单'"
      width="500px"
    >
      <el-form
        ref="formRef"
        :model="form"
        :rules="rules"
        label-width="80px"
      >
        <el-form-item label="菜单名称" prop="name">
          <el-input v-model="form.name" placeholder="请输入菜单名称" />
        </el-form-item>
        <el-form-item label="路径" prop="path">
          <el-input v-model="form.path" placeholder="请输入路径" />
        </el-form-item>
        <el-form-item label="组件" prop="component">
          <el-input v-model="form.component" placeholder="请输入组件路径" />
        </el-form-item>
        <el-form-item label="图标" prop="icon">
          <el-select v-model="form.icon" placeholder="请选择图标">
            <el-option label="仪表盘" value="Odometer" />
            <el-option label="用户" value="User" />
            <el-option label="锁定" value="Lock" />
            <el-option label="菜单" value="Menu" />
            <el-option label="设置" value="Setting" />
          </el-select>
        </el-form-item>
        <el-form-item label="排序" prop="sort">
          <el-input-number v-model="form.sort" :min="0" :max="999" />
        </el-form-item>
        <el-form-item label="状态" prop="status">
          <el-switch
            v-model="form.status"
            :active-value="1"
            :inactive-value="0"
          />
        </el-form-item>
        <el-form-item label="上级菜单" prop="parentId">
          <el-select v-model="form.parentId" placeholder="请选择上级菜单">
            <el-option :value="0" label="顶级菜单" />
            <el-option 
              v-for="item in parentMenuOptions" 
              :key="item.id" 
              :label="item.name" 
              :value="item.id" 
            />
          </el-select>
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
import { ref, reactive, computed } from 'vue'
import { ElMessage, ElMessageBox } from 'element-plus'
import { Search, Plus, Edit, Delete, Setting, Odometer, User, Lock, Menu as MenuIcon } from '@element-plus/icons-vue'
import type { FormInstance, FormRules } from 'element-plus'

// 搜索相关
const searchQuery = ref('')

// 分页相关
const currentPage = ref(1)
const pageSize = ref(10)
const total = ref(0)

// 表格数据
const menuList = ref([
  {
    id: 1,
    name: '仪表盘',
    path: '/dashboard',
    component: 'views/dashboard/index',
    icon: 'Odometer',
    sort: 1,
    status: 1,
    parentId: 0
  },
  {
    id: 2,
    name: '用户管理',
    path: '/user',
    component: 'views/user/index',
    icon: 'User',
    sort: 2,
    status: 1,
    parentId: 0
  },
  {
    id: 3,
    name: '角色管理',
    path: '/role',
    component: 'views/role/index',
    icon: 'Lock',
    sort: 3,
    status: 1,
    parentId: 0
  },
  {
    id: 4,
    name: '菜单管理',
    path: '/menu',
    component: 'views/menu/index',
    icon: 'Menu',
    sort: 4,
    status: 1,
    parentId: 0
  }
])

// 上级菜单选项
const parentMenuOptions = computed(() => {
  return menuList.value.filter(item => item.parentId === 0)
})

// 表单相关
const dialogVisible = ref(false)
const dialogType = ref<'add' | 'edit'>('add')
const formRef = ref<FormInstance>()

const form = reactive({
  id: '',
  name: '',
  path: '',
  component: '',
  icon: '',
  sort: 0,
  status: 1,
  parentId: 0
})

const rules = reactive<FormRules>({
  name: [
    { required: true, message: '请输入菜单名称', trigger: 'blur' }
  ],
  path: [
    { required: true, message: '请输入路径', trigger: 'blur' }
  ],
  component: [
    { required: true, message: '请输入组件路径', trigger: 'blur' }
  ],
  icon: [
    { required: true, message: '请选择图标', trigger: 'change' }
  ],
  sort: [
    { required: true, message: '请输入排序', trigger: 'blur' }
  ]
})

// 搜索处理
const handleSearch = () => {
  // TODO: 实现搜索逻辑
  console.log('搜索关键词:', searchQuery.value)
}

// 分页处理
const handleSizeChange = (val: number) => {
  pageSize.value = val
  // TODO: 重新加载数据
}

const handleCurrentChange = (val: number) => {
  currentPage.value = val
  // TODO: 重新加载数据
}

// 添加菜单
const handleAdd = () => {
  dialogType.value = 'add'
  dialogVisible.value = true
  // 重置表单
  Object.assign(form, {
    id: '',
    name: '',
    path: '',
    component: '',
    icon: '',
    sort: 0,
    status: 1,
    parentId: 0
  })
}

// 编辑菜单
const handleEdit = (row: any) => {
  dialogType.value = 'edit'
  dialogVisible.value = true
  // 填充表单数据
  Object.assign(form, row)
}

// 删除菜单
const handleDelete = (row: any) => {
  ElMessageBox.confirm(
    '确认删除该菜单吗？',
    '警告',
    {
      confirmButtonText: '确定',
      cancelButtonText: '取消',
      type: 'warning',
    }
  )
    .then(() => {
      // TODO: 实现删除逻辑
      ElMessage({
        type: 'success',
        message: '删除成功',
      })
    })
    .catch(() => {
      // 取消删除
    })
}

// 提交表单
const handleSubmit = async () => {
  if (!formRef.value) return
  
  await formRef.value.validate((valid, fields) => {
    if (valid) {
      // TODO: 实现提交逻辑
      console.log('表单数据:', form)
      ElMessage({
        type: 'success',
        message: dialogType.value === 'add' ? '添加成功' : '更新成功'
      })
      dialogVisible.value = false
    } else {
      console.error('表单验证失败:', fields)
    }
  })
}
</script>

<style scoped>
.menu-container {
  padding: 20px;
}

.search-bar {
  margin-bottom: 20px;
  display: flex;
  gap: 16px;
}

.search-input {
  width: 300px;
}

.pagination {
  margin-top: 20px;
  display: flex;
  justify-content: flex-end;
}
</style>