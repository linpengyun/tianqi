<template>
  <div class="role-container">
    <el-card class="box-card">
      <template #header>
        <div class="card-header">
          <span>角色列表</span>
          <el-button type="primary" @click="handleAdd">新增角色</el-button>
        </div>
      </template>

      <el-table :data="roleList" style="width: 100%" v-loading="loading">
        <el-table-column prop="roleName" label="角色名称" />
        <el-table-column prop="roleKey" label="角色标识" />
        <el-table-column prop="description" label="描述" />
        <el-table-column prop="createTime" label="创建时间" />
        <el-table-column label="操作" width="200">
          <template #default="scope">
            <el-button type="primary" size="small" @click="handleEdit(scope.row)">编辑</el-button>
            <el-button type="success" size="small" @click="handlePermission(scope.row)">权限</el-button>
            <el-button type="danger" size="small" @click="handleDelete(scope.row)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>

      <!-- 分页 -->
      <div class="pagination-container">
        <el-pagination
          v-model:current-page="currentPage"
          v-model:page-size="pageSize"
          :page-sizes="[10, 20, 30, 50]"
          layout="total, sizes, prev, pager, next, jumper"
          :total="total"
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
        />
      </div>
    </el-card>

    <!-- 角色表单对话框 -->
    <el-dialog
      :title="dialogTitle"
      v-model="dialogVisible"
      width="500px"
      @close="handleDialogClose"
    >
      <el-form
        ref="roleFormRef"
        :model="roleForm"
        :rules="rules"
        label-width="100px"
      >
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="roleForm.roleName" placeholder="请输入角色名称" />
        </el-form-item>
        <el-form-item label="角色标识" prop="roleKey">
          <el-input v-model="roleForm.roleKey" placeholder="请输入角色标识" />
        </el-form-item>
        <el-form-item label="角色描述" prop="description">
          <el-input
            v-model="roleForm.description"
            type="textarea"
            placeholder="请输入角色描述"
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

    <!-- 权限分配对话框 -->
    <el-dialog
      title="分配权限"
      v-model="permissionDialogVisible"
      width="600px"
    >
      <el-tree
        ref="permissionTreeRef"
        :data="permissionTree"
        show-checkbox
        node-key="id"
        :props="{
          children: 'children',
          label: 'name'
        }"
      />
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="permissionDialogVisible = false">取消</el-button>
          <el-button type="primary" @click="handlePermissionSubmit">确定</el-button>
        </span>
      </template>
    </el-dialog>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive } from 'vue'
import { ElMessage, ElMessageBox } from 'element-plus'
import type { FormInstance, FormRules } from 'element-plus'

// 表格数据
const loading = ref(false)
const roleList = ref([])
const currentPage = ref(1)
const pageSize = ref(10)
const total = ref(0)

// 表单数据
const dialogVisible = ref(false)
const dialogTitle = ref('')
const roleFormRef = ref<FormInstance>()
const roleForm = reactive({
  id: '',
  roleName: '',
  roleKey: '',
  description: ''
})

// 表单验证规则
const rules = reactive<FormRules>({
  roleName: [
    { required: true, message: '请输入角色名称', trigger: 'blur' },
    { min: 2, max: 20, message: '长度在 2 到 20 个字符', trigger: 'blur' }
  ],
  roleKey: [
    { required: true, message: '请输入角色标识', trigger: 'blur' },
    { min: 2, max: 20, message: '长度在 2 到 20 个字符', trigger: 'blur' }
  ]
})

// 权限树数据
const permissionDialogVisible = ref(false)
const permissionTreeRef = ref()
const permissionTree = ref([])
const currentRole = ref({})

// 获取角色列表
const getRoleList = async () => {
  loading.value = true
  try {
    // TODO: 调用获取角色列表接口
    // const res = await getRoles({
    //   pageNum: currentPage.value,
    //   pageSize: pageSize.value
    // })
    // roleList.value = res.data.list
    // total.value = res.data.total

    // 模拟数据
    roleList.value = [
      {
        id: 1,
        roleName: '管理员',
        roleKey: 'admin',
        description: '系统管理员',
        createTime: '2023-01-01'
      }
    ]
    total.value = 1
  } catch (error) {
    console.error('获取角色列表失败:', error)
    ElMessage.error('获取角色列表失败')
  } finally {
    loading.value = false
  }
}

// 分页处理
const handleSizeChange = (val: number) => {
  pageSize.value = val
  getRoleList()
}

const handleCurrentChange = (val: number) => {
  currentPage.value = val
  getRoleList()
}

// 新增角色
const handleAdd = () => {
  dialogTitle.value = '新增角色'
  dialogVisible.value = true
  roleForm.id = ''
  roleForm.roleName = ''
  roleForm.roleKey = ''
  roleForm.description = ''
}

// 编辑角色
const handleEdit = (row: any) => {
  dialogTitle.value = '编辑角色'
  dialogVisible.value = true
  roleForm.id = row.id
  roleForm.roleName = row.roleName
  roleForm.roleKey = row.roleKey
  roleForm.description = row.description
}

// 删除角色
const handleDelete = (row: any) => {
  ElMessageBox.confirm('确认删除该角色吗？', '提示', {
    confirmButtonText: '确定',
    cancelButtonText: '取消',
    type: 'warning'
  })
    .then(async () => {
      try {
        // TODO: 调用删除角色接口
        // await deleteRole(row.id)
        ElMessage.success('删除成功')
        getRoleList()
      } catch (error) {
        console.error('删除角色失败:', error)
        ElMessage.error('删除角色失败')
      }
    })
    .catch(() => {
      // 取消删除
    })
}

// 提交表单
const handleSubmit = async () => {
  if (!roleFormRef.value) return
  await roleFormRef.value.validate(async (valid) => {
    if (valid) {
      try {
        // TODO: 调用新增/编辑角色接口
        // if (roleForm.id) {
        //   await updateRole(roleForm)
        // } else {
        //   await addRole(roleForm)
        // }
        ElMessage.success(roleForm.id ? '编辑成功' : '新增成功')
        dialogVisible.value = false
        getRoleList()
      } catch (error) {
        console.error('保存角色失败:', error)
        ElMessage.error('保存角色失败')
      }
    }
  })
}

// 关闭对话框
const handleDialogClose = () => {
  if (!roleFormRef.value) return
  roleFormRef.value.resetFields()
}

// 打开权限分配对话框
const handlePermission = (row: any) => {
  currentRole.value = row
  permissionDialogVisible.value = true
  // TODO: 获取权限树数据
  // getPermissionTree()
  // 模拟数据
  permissionTree.value = [
    {
      id: 1,
      name: '系统管理',
      children: [
        {
          id: 2,
          name: '用户管理'
        },
        {
          id: 3,
          name: '角色管理'
        }
      ]
    }
  ]
}

// 提交权限分配
const handlePermissionSubmit = async () => {
  try {
    const checkedKeys = permissionTreeRef.value.getCheckedKeys()
    // TODO: 调用分配权限接口
    // await assignPermissions({
    //   roleId: currentRole.value.id,
    //   permissionIds: checkedKeys
    // })
    ElMessage.success('权限分配成功')
    permissionDialogVisible.value = false
  } catch (error) {
    console.error('权限分配失败:', error)
    ElMessage.error('权限分配失败')
  }
}

// 初始化
getRoleList()
</script>

<style scoped>
.role-container {
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
</style>