<template>
  <container>
    <!--查询-->
    <template #header>
      <el-form class="ad-form-query" :inline="true" :model="filter" @submit.native.prevent>
        <el-form-item>
          <el-input
            v-model="filter.userName"
            placeholder="用户名/昵称"
            clearable
            @keyup.enter.native="getUsers"
          >
            <template #prefix>
              <i class="el-input__icon el-icon-search" />
            </template>
          </el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="getUsers">查询</el-button>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="onAdd">新增</el-button>
        </el-form-item>
        <el-form-item>
          <confirm-button
            :disabled="sels.length === 0"
            :type="'delete'"
            :placement="'bottom-end'"
            :loading="deleteLoading"
            :validate="batchDeleteValidate"
            style="margin-left: 0px;"
            @click="onBatchDelete"
          >
            <template #content>
              <p>确定要批量删除吗？</p>
            </template>
            批量删除
          </confirm-button>
        </el-form-item>
      </el-form>
    </template>

    <!--列表-->
    <el-table
      v-loading="listLoading"
      :data="users"
      highlight-current-row
      height="'100%'"
      style="width: 100%;height:100%;"
      @selection-change="onSelsChange"
    >
      <el-table-column type="selection" width="50" />
      <el-table-column type="index" width="80" label="#" />
      <el-table-column prop="userName" label="用户名" width />
      <el-table-column prop="nickName" label="昵称" width />
      <el-table-column prop="roleNames" label="角色" width>
        <template v-slot="{row}">
          {{ row.roleNames ? row.roleNames.join(','):'' }}
        </template>
      </el-table-column>
      <el-table-column prop="createdTime" label="创建时间" :formatter="formatCreatedTime" width />
      <el-table-column prop="status" label="状态" width>
        <template v-slot="{row}">
          <el-tag :type="row.status == 0 ? 'success' : 'danger'" disable-transitions>
            {{ row.status == 0 ? '正常' : '禁用' }}
          </el-tag>
        </template>
      </el-table-column>
      <el-table-column label="操作" width="180">
        <template v-slot="{ $index, row }">
          <el-button @click="onEdit($index, row)">编辑</el-button>
          <confirm-button
            type="delete"
            :loading="row._loading"
            :validate="deleteValidate"
            :validate-data="row"
            @click="onDelete($index, row)"
          />
        </template>
      </el-table-column>
    </el-table>

    <!--分页-->
    <template #footer>
      <pagination
        ref="pager"
        :page.sync="pager.currentPage"
        :size.sync="pager.pageSize"
        :total="pager.total"
        :checked-count="sels.length"
        @get-page="getUsers"
      />
    </template>

    <!--新增窗口-->
    <el-drawer
      title="新增用户"
      :wrapper-closable="true"
      :visible.sync="addFormVisible"
      direction="btt"
      size="auto"
      class="el-drawer__wrapper"
      @close="closeAddForm"
    >
      <section style="padding:24px 48px 74px 24px;">
        <el-form
          ref="addForm"
          :model="addForm"
          :rules="addFormRules"
          label-width="80px"
          :inline="false"
        >
          <el-row>
            <el-col :xs="24" :sm="24" :md="24" :lg="18" :xl="18">
              <el-col :xs="24" :sm="12" :md="8" :lg="8" :xl="6">
                <el-form-item label="用户名" prop="userName">
                  <el-input
                    v-model="addForm.userName"
                    autocomplete="off"
                    :readonly="userNameReadonly"
                    @focus="userNameReadonly = false"
                    @blur="userNameReadonly = true"
                  />
                </el-form-item>
              </el-col>
              <el-col :xs="24" :sm="12" :md="8" :lg="8" :xl="6">
                <el-form-item label="密码" prop="password">
                  <el-input v-model="addForm.password" show-password autocomplete="off" />
                </el-form-item>
              </el-col>
              <el-col :xs="24" :sm="12" :md="8" :lg="8" :xl="6">
                <el-form-item label="昵称" prop="nickName">
                  <el-input v-model="addForm.nickName" autocomplete="off" />
                </el-form-item>
              </el-col>
              <el-col :xs="24" :sm="12" :md="8" :lg="8" :xl="6">
                <el-form-item label="角色" prop="roleIds">
                  <el-select v-model="addForm.roleIds" multiple placeholder="请选择角色" style="width:100%;">
                    <el-option
                      v-for="item in roles"
                      :key="item.id"
                      :label="item.name"
                      :value="item.id"
                    />
                  </el-select>
                </el-form-item>
              </el-col>
            </el-col>
          </el-row>
        </el-form>
      </section>
      <div class="drawer-footer">
        <el-button @click.native="addFormVisible = false">取消</el-button>
        <confirm-button type="submit" :validate="addFormvalidate" :loading="addLoading" @click="onAddSubmit" />
      </div>
    </el-drawer>

    <!--编辑窗口-->
    <el-drawer
      title="编辑用户"
      :wrapper-closable="true"
      :close-on-press-escape="true"
      :visible.sync="editFormVisible"
      direction="btt"
      size="'auto'"
      class="el-drawer__wrapper"
      @close="closeEditForm"
    >
      <section style="padding:24px 48px 74px 24px;">
        <el-form
          ref="editForm"
          :model="editForm"
          :rules="editFormRules"
          label-width="80px"
          :inline="false"
        >
          <el-row>
            <el-col :xs="24" :sm="24" :md="24" :lg="18" :xl="18">
              <el-col :xs="24" :sm="12" :md="8" :lg="8" :xl="6">
                <el-form-item label="用户名" prop="userName">
                  <el-input v-model="editForm.userName" autocomplete="off" />
                </el-form-item>
              </el-col>
              <el-col :xs="24" :sm="12" :md="8" :lg="8" :xl="6">
                <el-form-item label="昵称" prop="nickName">
                  <el-input v-model="editForm.nickName" autocomplete="off" />
                </el-form-item>
              </el-col>
              <el-col :xs="24" :sm="12" :md="8" :lg="8" :xl="6">
                <el-form-item label="角色" prop="roleIds">
                  <el-select v-model="editForm.roleIds" multiple placeholder="请选择角色" style="width:100%;">
                    <el-option
                      v-for="item in roles"
                      :key="item.id"
                      :label="item.name"
                      :value="item.id"
                    />
                  </el-select>
                </el-form-item>
              </el-col>
            </el-col>
          </el-row>
        </el-form>
      </section>
      <div class="drawer-footer">
        <el-button @click.native="editFormVisible = false">取消</el-button>
        <confirm-button type="submit" :validate="editFormvalidate" :loading="editLoading" @click="onEditSubmit" />
      </div>
    </el-drawer>
  </container>
</template>

<script>
import { formatTime } from '@/utils'
import { getRoleListPage } from '@/api/admin/role'
import { getUserListPage, removeUser, batchRemoveUser, editUser, addUser, getUser } from '@/api/admin/user'
import Container from '@/components/Container'
import ConfirmButton from '@/components/ConfirmButton'
import Pagination from '@/components/Pagination'

export default {
  name: 'Users',
  components: { Container, ConfirmButton, Pagination },
  data() {
    return {
      filter: {
        userName: ''
      },
      users: [],
      roles: [],
      pager: {},
      listLoading: false,
      sels: [], // 列表选中列

      addDialogFormVisible: false,
      editFormVisible: false, // 编辑界面是否显示
      editLoading: false,
      editFormRules: {
        userName: [{ required: true, message: '请输入用户名', trigger: 'blur' }]
      },
      userNameReadonly: true,
      // 编辑界面数据
      editForm: {
        id: 0,
        userName: '',
        nickName: '',
        roleIds: []
      },

      addFormVisible: false, // 新增界面是否显示
      addLoading: false,
      addFormRules: {
        userName: [{ required: true, message: '请输入用户名', trigger: 'blur' }],
        password: [{ required: true, message: '请输入密码', trigger: 'blur' }]
      },
      // 新增界面数据
      addForm: {
        userName: '',
        nickName: '',
        password: '',
        roleIds: []
      },
      deleteLoading: false
    }
  },
  async mounted() {
    this.pager = this.$refs.pager.getPager()
    this.getUsers()
  },
  methods: {
    formatCreatedTime(row, column, time) {
      return formatTime(time, 'yyyy-MM-dd hh:mm')
    },
    // 获取用户列表
    async getUsers() {
      const para = {
        currentPage: this.pager.currentPage,
        pageSize: this.pager.pageSize,
        filter: this.filter
      }

      this.listLoading = true
      const res = await getUserListPage(para)
      this.listLoading = false

      if (!res.success) {
        if (res.msg) {
          this.$message({
            message: res.msg,
            type: 'error'
          })
        }
        return
      }

      this.pager.total = res.data.total
      const data = res.data.list
      data.forEach(d => {
        d._loading = false
      })
      this.users = data
    },
    async getRoleListPage() {
      const res = await getRoleListPage()
      if (res && res.success) {
        this.roles = res.data.list
      }
    },
    // 显示编辑界面
    async onEdit(index, row) {
      const loading = this.$loading()
      if (this.roles.length === 0) {
        await this.getRoleListPage()
      }
      const res = await getUser({ id: row.id })
      loading.close()
      if (res && res.success) {
        const data = res.data
        this.editForm = data
        this.editFormVisible = true
      }
    },
    closeEditForm() {
      this.$refs.editForm.resetFields()
    },
    // 显示新增界面
    async onAdd() {
      if (this.roles.length === 0) {
        const loading = this.$loading()
        await this.getRoleListPage()
        loading.close()
      }
      this.addFormVisible = true
    },
    closeAddForm() {
      this.$refs.addForm.resetFields()
    },
    // 编辑验证
    editFormvalidate() {
      let isValid = false
      this.$refs.editForm.validate(valid => {
        isValid = valid
      })
      return isValid
    },
    // 编辑
    async onEditSubmit() {
      this.editLoading = true
      const para = _.cloneDeep(this.editForm)

      const res = await editUser(para)
      this.editLoading = false

      if (res.success) {
        this.$message({
          message: this.$t('admin.updateOk'),
          type: 'success'
        })
        this.$refs['editForm'].resetFields()
        this.editFormVisible = false
        this.getUsers()
      } else {
        this.$message({
          message: res.msg,
          type: 'error'
        })
      }
    },
    // 新增验证
    addFormvalidate() {
      let isValid = false
      this.$refs.addForm.validate(valid => {
        isValid = valid
      })
      return isValid
    },
    // 新增
    async onAddSubmit() {
      this.addLoading = true
      const para = _.cloneDeep(this.addForm)

      const res = await addUser(para)
      this.addLoading = false

      if (res.success) {
        this.$message({
          message: this.$t('admin.addOk'),
          type: 'success'
        })
        this.$refs['addForm'].resetFields()
        this.addFormVisible = false
        this.getUsers()
      } else {
        this.$message({
          message: res.msg,
          type: 'error'
        })
      }
    },
    // 删除验证
    deleteValidate(row) {
      let isValid = true
      if (row && row.userName === 'admin') {
        this.$message({
          message: row.nickName + '，禁止删除！',
          type: 'warning'
        })
        isValid = false
      }

      return isValid
    },
    // 删除
    async onDelete(index, row) {
      row._loading = true
      const para = { id: row.id }
      const res = await removeUser(para)
      row._loading = false

      if (!res.success) {
        this.$message({
          message: res.msg,
          type: 'error'
        })
        return
      }
      this.$message({
        message: this.$t('admin.deleteOk'),
        type: 'success'
      })
      this.getUsers()
    },
    // 批量删除验证
    batchDeleteValidate() {
      let isValid = true
      var row = this.sels && this.sels.find(s => s.userName === 'admin')
      if (row && row.userName === 'admin') {
        this.$message({
          message: row.nickName + '，禁止删除！',
          type: 'warning'
        })
        isValid = false
      }

      return isValid
    },
    // 批量删除
    async onBatchDelete() {
      const para = { ids: [] }
      para.ids = this.sels.map(s => {
        return s.id
      })

      this.deleteLoading = true
      const res = await batchRemoveUser(para.ids)
      this.deleteLoading = false

      if (!res.success) {
        this.$message({
          message: res.msg,
          type: 'error'
        })
        return
      }
      this.$message({
        message: this.$t('admin.batchDeleteOk'),
        type: 'success'
      })

      this.getUsers()
    },
    // 选择
    onSelsChange(sels) {
      this.sels = sels
    }
  }
}
</script>
