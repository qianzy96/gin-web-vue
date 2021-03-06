<template>
  <div class="app-container">
    <el-form
      ref="searchForm"
      v-loading="table.loading"
      :inline="true"
      :model="table.form"
      class="demo-form-inline"
    >
      <el-form-item
        label="用户名"
        prop="username"
      >
        <el-input
          v-model.trim="table.form.username"
          placeholder="用户名"
          clearable
          @clear="doSearch"
          @keyup.enter.native="doSearch"
        />
      </el-form-item>
      <el-form-item
        label="手机号"
        prop="mobile"
      >
        <el-input
          v-model.trim="table.form.mobile"
          placeholder="手机号"
          clearable
          @clear="doSearch"
          @keyup.enter.native="doSearch"
        />
      </el-form-item>
      <el-form-item
        label="昵称"
        prop="nickname"
      >
        <el-input
          v-model.trim="table.form.nickname"
          placeholder="昵称"
          clearable
          @clear="doSearch"
          @keyup.enter.native="doSearch"
        />
      </el-form-item>
      <el-form-item
        label="状态"
        prop="status"
      >
        <el-select
          v-model.trim="table.form.status"
          clearable
          placeholder="用户状态"
        >
          <el-option
            key="true"
            label="正常"
            :value="true"
          />
          <el-option
            key="false"
            label="禁用"
            :value="false"
          />
        </el-select>
      </el-form-item>
      <el-form-item
        label="创建人"
        prop="creator"
      >
        <el-input
          v-model.trim="table.form.creator"
          placeholder="创建人"
          clearable
          @clear="doSearch"
          @keyup.enter.native="doSearch"
        />
      </el-form-item>
      <el-form-item>
        <el-button
          type="primary"
          :loading="table.loading"
          @click="doSearch"
        >
          查询
        </el-button>
        <el-button @click="resetForm('searchForm')">
          重置
        </el-button>
      </el-form-item>
      <el-row :gutter="10">
        <el-col :span="1.5">
          <el-button
            type="primary"
            icon="el-icon-plus"
            @click="handleCreate"
          >
            新增
          </el-button>
        </el-col>
        <el-col :span="1.5">
          <el-button
            type="danger"
            icon="el-icon-delete"
            :disabled="table.batchDeleteBtnDisabled"
            @click="handleBatchDelete"
          >
            批量删除
          </el-button>
        </el-col>
      </el-row>
      <el-table
        :data="table.list"
        style="width: 100%;margin-bottom: 20px;"
        row-key="id"
        border
        stripe
        @selection-change="handleSelectionChange"
      >
        <el-table-column
          type="selection"
          width="55"
        />
        <el-table-column
          prop="username"
          label="用户名"
          sortable
        />
        <el-table-column
          prop="mobile"
          label="手机号"
          sortable
        />
        <el-table-column
          prop="nickname"
          label="昵称"
        />
        <el-table-column
          prop="status"
          label="状态"
        >
          <template slot-scope="scope">
            <el-switch
              v-model.trim="scope.row.status"
              @change="handleStatusChange(scope.row)"
            />
          </template>
        </el-table-column>
        <el-table-column
          prop="creator"
          label="创建人"
        />
        <el-table-column
          fixed="right"
          label="操作"
          align="center"
          width="180"
        >
          <template slot-scope="scope">
            <el-button
              size="mini"
              @click="handleUpdate(scope.row)"
            >
              编辑
            </el-button>
            <el-button
              size="mini"
              type="danger"
              @click="handleDelete(scope.row)"
            >
              删除
            </el-button>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination
        :current-page="table.pageNum"
        :page-sizes="[1, 5, 20, 50]"
        :page-size="table.pageSize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="table.total"
        background
        @size-change="handlePagSizeChange"
        @current-change="handlePageNumChange"
      />
    </el-form>

    <!-- 用户配置对话框 -->
    <el-dialog
      :title="updateDialog.title"
      :visible.sync="updateDialog.visible"
      width="500px"
    >
      <el-form
        ref="updateForm"
        :model="updateDialog.form"
        :rules="updateDialog.rules"
        label-width="80px"
      >
        <el-form-item
          label="用户名"
          prop="username"
        >
          <el-input
            v-model.trim="updateDialog.form.username"
            placeholder="请输入用户名(用于登录系统)"
          />
        </el-form-item>
        <el-form-item
          v-if="updateDialog.type===0"
          label="初始密码"
          prop="initPassword"
        >
          <el-input
            v-model.trim="updateDialog.form.initPassword"
            placeholder="请输入用户初始密码(用于登录系统)"
          />
        </el-form-item>
        <el-form-item
          label="手机号"
          prop="mobile"
        >
          <el-input
            v-model.trim="updateDialog.form.mobile"
            placeholder="请输入手机号"
          />
        </el-form-item>
        <el-form-item
          label="绑定角色"
          prop="roleId"
        >
          <el-select
            v-model.trim="updateDialog.form.roleId"
            clearable
            filterable
            placeholder="请为该用户指定角色"
          >
            <el-option
              v-for="item in updateDialog.roleSelectOptions"
              :key="item.id"
              :label="item.name + '(' + item.desc + ')'"
              :value="item.id"
            />
          </el-select>
        </el-form-item>
        <el-form-item
          v-if="updateDialog.type===1"
          label="重置密码"
          prop="newPassword"
        >
          <el-input
            v-model.trim="updateDialog.form.newPassword"
            placeholder="请输入新的用户密码"
          />
        </el-form-item>
        <el-form-item
          label="昵称"
          prop="nickname"
        >
          <el-input
            v-model.trim="updateDialog.form.nickname"
            placeholder="请输入昵称"
          />
        </el-form-item>
        <el-form-item
          label="状态"
          prop="status"
        >
          <el-switch
            v-model.trim="updateDialog.form.status"
            active-text="正常"
            inactive-text="禁用"
          />
        </el-form-item>
      </el-form>
      <div
        slot="footer"
        class="dialog-footer"
      >
        <el-button
          type="primary"
          :loading="updateDialog.loading"
          @click="doUpdate"
        >
          确 定
        </el-button>
        <el-button @click="cancelUpdate">
          取 消
        </el-button>
      </div>
    </el-dialog>
  </div>
</template>
<script lang="ts">
import { Component, Vue } from 'vue-property-decorator'
import Pagination from '@/components/Pagination/index.vue'
import { Form } from 'element-ui'
import { batchDeleteUser, createUser, getUsers, updateUser } from '@/api/system/users'
import { diffObjUpdate } from '@/utils/diff'
import { getRoles } from '@/api/system/roles'

@Component({
  // 组件名称首字母需大写, 否则会报警告
  name: 'User',
  components: {
    Pagination
  }
})
export default class extends Vue {
  private readonly defaultConfig: any = {
    pageNum: 1,
    pageSize: 5
  }

  private validateUsername(rule: any, value: string, callback: Function) {
    if (!/^[a-zA-Z]/.test(value)) {
      callback(new Error('必须以字母开头, 如a12345'))
    } else if (!/^[a-zA-Z]([-_a-zA-Z0-9])+$/.test(value)) {
      callback(new Error('不允许出现汉字或特殊字符, 如a+,sa、a张三'))
    } else {
      callback()
    }
  }

  private validateMobile(rule: any, value: string, callback: Function) {
    if (!/^[1-9]([0-9]{10})+$/.test(value)) {
      callback(new Error('手机号格式不正确'))
    } else {
      callback()
    }
  }

  private updateDialog: any = {
    loading: false,
    // 类型(0:创建, 1更新)
    type: 0,
    // 是否打开
    visible: false,
    // 标题
    title: '',
    // 角色选择参数
    roleSelectOptions: [],
    // 默认数据
    defaultForm: {
      id: 0,
      username: '',
      initPassword: '',
      newPassword: '',
      mobile: '',
      nickname: '',
      status: true,
      creator: '',
      roleId: ''
    },
    // 表单
    form: {},
    oldData: {},
    // 表单校验
    rules: {
      username: [
        { required: true, message: '用户名不能为空', trigger: 'blur' },
        { min: 4, message: '用户名至少4个字符', trigger: 'blur' },
        { max: 20, message: '用户名至多20个字符', trigger: 'blur' },
        { validator: this.validateUsername, trigger: 'blur' }
      ],
      initPassword: [
        { required: true, message: '用户初始密码不能为空', trigger: 'blur' },
        { min: 6, message: '用户初始密码至少6个字符', trigger: 'blur' }
      ],
      mobile: [
        { required: true, message: '手机号不能为空', trigger: 'blur' },
        { validator: this.validateMobile, trigger: 'blur' }
      ],
      roleId: [
        { required: true, message: '请为用户绑定角色', trigger: 'blur' }
      ],
      newPassword: [
        { min: 6, message: '重置用户密码至少6个字符', trigger: 'blur' }
      ]
    }
  }

  private table: any = {
    loading: false,
    batchDeleteBtnDisabled: true,
    selection: [],
    list: [],
    pageNum: 1,
    pageSize: 5,
    total: 0,
    form: {
      username: '',
      mobile: '',
      nickname: '',
      status: '',
      creator: ''
    }
  }

  created() {
    this.resetUpdateForm()
    this.getData()
  }

  private async getData() {
    this.table.loading = true
    try {
      const params: any = {
        ...this.table.form,
        pageNum: this.table.pageNum,
        pageSize: this.table.pageSize
      }
      if (params.status === '') {
        delete params.status
      }
      const { data } = await getUsers(params)
      this.table.list = data.list
      this.table.pageNum = data.pageNum
      this.table.pageSize = data.pageSize
      this.table.total = data.total
    } finally {
      this.table.loading = false
    }
  }

  private async doSearch() {
    this.getData()
  }

  private async doUpdate() {
    (this.$refs.updateForm as Form).validate(async(valid) => {
      if (valid) {
        if (this.updateDialog.type === 0) {
          try {
            this.updateDialog.loading = true
            await createUser(this.updateDialog.form)
          } finally {
            this.updateDialog.loading = false
          }
        } else {
          const update = diffObjUpdate(this.updateDialog.oldData, this.updateDialog.form)
          // 编号存在则更新, 不存在给出提示
          if (!update.id) {
            this.$message({
              type: 'warning',
              message: '数据没有发生变化, 请重新输入~'
            })
            return
          }
          try {
            this.updateDialog.loading = true
            await updateUser(update.id, update)
          } finally {
            this.updateDialog.loading = false
          }
        }
        // 关闭弹窗
        this.updateDialog.visible = false
        // 重新查询
        this.getData()
        // 清理字段
        this.resetUpdateForm()
        this.$notify({
          title: '恭喜',
          message: `${this.updateDialog.type === 0 ? '创建' : '更新'}用户成功`,
          type: 'success',
          duration: 2000
        })
      }
    })
  }

  private async resetUpdateForm() {
    this.$nextTick(() => {
      // 重置校验信息
      const form = this.$refs.updateForm as Form
      if (form) {
        form.clearValidate()
      }
    })
    this.updateDialog.form = JSON.parse(JSON.stringify(this.updateDialog.defaultForm))
  }

  private async cancelUpdate() {
    // 关闭弹窗
    this.updateDialog.visible = false
  }

  private async handleCreate() {
    // 之前的类型为更新
    if (this.updateDialog.type === 1) {
      // 清理字段
      this.resetUpdateForm()
    }
    try {
      // 读取所有角色
      const { data } = await getRoles({
        noPagination: true
      })
      this.updateDialog.roleSelectOptions = data.list
    } catch (e) {
      this.$message.error('读取系统角色信息失败')
      return
    }
    // 修改类型
    this.updateDialog.type = 0
    // 修改标题
    this.updateDialog.title = '创建新用户'
    // 开启弹窗
    this.updateDialog.visible = true
  }

  private async handleUpdate(row: any) {
    try {
      // 读取当前角色
      const { data } = await getRoles({
        noPagination: true
      })
      this.updateDialog.roleSelectOptions = data.list
    } catch (e) {
      this.$message.error('读取系统角色信息失败')
      return
    }
    // 清理字段
    this.resetUpdateForm()
    // 弹窗表单赋值
    this.updateDialog.form.id = row.id
    this.updateDialog.form.username = row.username
    this.updateDialog.form.mobile = row.mobile
    this.updateDialog.form.nickname = row.nickname
    this.updateDialog.form.status = row.status
    this.updateDialog.form.roleId = row.roleId
    // 记录旧数据
    this.updateDialog.oldData = JSON.parse(JSON.stringify(this.updateDialog.form))
    // 修改类型
    this.updateDialog.type = 1
    // 修改标题
    this.updateDialog.title = '修改用户信息'
    // 开启弹窗
    this.updateDialog.visible = true
  }

  private handleDelete(row: any) {
    this.batchDelete([row])
  }

  private handleBatchDelete() {
    this.batchDelete(this.table.selection)
  }

  private async batchDelete(rows: any) {
    const ids: number[] = []
    const usernames: string[] = []
    for (let i = 0, len = rows.length; i < len; i++) {
      const row = rows[i]
      ids.push(row.id)
      usernames.push(row.username)
    }
    if (ids.length > 0) {
      const msg = `确定要删除用户[${usernames.join(',')}]吗, 此操作不可逆?`
      this.$confirm(msg, '请谨慎操作', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(async() => {
          await batchDeleteUser(ids.join(','))
          this.getData()
        })
    }
  }

  // 改变用户状态
  private async handleStatusChange(row: any) {
    let msg = `确定要恢复用户[${row.username}]吗?`
    if (!row.status) {
      msg = `确定要禁用用户[${row.username}]吗?该操作可能导致该用户无法正常使用系统`
    }
    this.$confirm(msg, '请谨慎操作', {
      confirmButtonText: '确定',
      cancelButtonText: '取消',
      type: 'warning'
    })
      .then(async() => {
        await updateUser(row.id, {
          status: row.status
        })
        this.getData()
      })
      .catch(() => {
        row.status = !row.status
      })
  }

  private resetForm(formName: string) {
    // 重置分页
    this.table.pageNum = this.defaultConfig.pageNum
    this.table.pageSize = this.defaultConfig.pageSize
    // 仅对el-form-item设置prop的字段有效
    // 这种写法可能导致无法正确执行: (this.$refs[formName] as Form).resetFields()
    const form = this.$refs[formName] as Form
    form.resetFields()
    // 重新获取数据
    this.getData()
  }

  private async handleSelectionChange(rows: any) {
    this.table.batchDeleteBtnDisabled = rows.length === 0
    this.table.selection = rows
  }

  private handlePagSizeChange(val: number) {
    this.table.pageSize = val
    this.doSearch()
  }

  private handlePageNumChange(val: number) {
    this.table.pageNum = val
    this.doSearch()
  }
}
</script>
<style lang="scss" scoped>
  .app-container {
    .el-row {
      margin-bottom: 15px;
    }
  }
</style>
