<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card class="box-card">
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input
            v-model="querycdt.query"
            placeholder="请输入搜索内容"
            :clearable="true"
            @keyup.enter.native="getUserInfos"
            @clear="getUserInfos"
          >
            <el-button slot="append" icon="el-icon-search" @click="getUserInfos"></el-button>
          </el-input>
        </el-col>
        <el-col :span="8">
          <el-button type="primary" @click="addDialogVisible=true">添加用户</el-button>
        </el-col>
      </el-row>
      <el-table :data="userInfo" border style="width: 100%">
        <el-table-column type="index" label="序号" width="60"></el-table-column>
        <el-table-column prop="username" label="用户名" width="200"></el-table-column>
        <el-table-column prop="mobile" label="手机号码" width="200"></el-table-column>
        <el-table-column prop="role_name" label="角色" width="200"></el-table-column>
        <el-table-column prop="email" label="邮箱" width="250"></el-table-column>
        <el-table-column prop="mg_state" label="状态" width="60">
          <el-switch
            v-model="info.row.mg_state"
            slot-scope="info"
            @change="changeState(info.row.id, info.row.mg_state)"
          ></el-switch>
        </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="info">
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="showEditDialog(info.row.id)"
            ></el-button>
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="delUser(info.row.id)"
            ></el-button>
            <el-tooltip content="分配角色" placement="top" :enterable="false">
              <el-button type="warning" icon="el-icon-setting" size="mini"></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination
        @size-change="handelSizeChange"
        @current-change="handelCurrentChange"
        :current-page="querycdt.pagenum"
        :page-sizes="[2, 5, 10]"
        :page-size="querycdt.pagesize"
        layout="prev, pager, next, sizes, total, jumper"
        :total="querycdt.total"
      ></el-pagination>
    </el-card>

    <el-dialog title="添加用户" :visible.sync="addDialogVisible" width="30%" @close="addDialogClose">
      <el-form ref="addFormRef" :rules="addFormRules" label-width="80px" :model="addForm">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="addForm.username"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="addForm.password"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="addForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机号码" prop="mobile">
          <el-input v-model="addForm.mobile"></el-input>
        </el-form-item>
      </el-form>

      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </span>
    </el-dialog>

    <el-dialog title="修改用户" :visible.sync="editDialogVisible" width="30%" @close="editDialogClose">
      <el-form ref="editFormRef" :rules="editFormRules" label-width="80px" :model="editForm">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="editForm.username" :disabled="true"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="editForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机号码" prop="mobile">
          <el-input v-model="editForm.mobile"></el-input>
        </el-form-item>
      </el-form>

      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editUser">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  created() {
    this.getUserInfos()
  },
  methods: {
    editUser() {
      this.$refs.editFormRef.validate(async valid => {
        if (valid) {
          const { data: res } = await this.$http.put(
            'users/' + this.editForm.id,
            this.editForm
          )
          if (res.meta.status !== 200) {
            return this.$message.error(res.meta.msg)
          }
          this.$message.success(res.meta.msg)
          this.editDialogVisible = false
          this.getUserInfos()
        }
      })
    },
    async showEditDialog(id) {
      this.editDialogVisible = true
      const { data: res } = await this.$http.get('users/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.editForm = res.data
    },
    editDialogClose(id) {
      this.$refs.editFormRef.resetFields()
    },
    delUser(id) {
      this.$confirm('确定删除吗?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(async() => {
          const { data: res } = await this.$http.delete('users/' + id)
          if (res.meta.status !== 200) {
            return this.$message.error(res.meta.msg)
          }
          this.$message.success(res.meta.msg)
          this.getUserInfos()
        })
        .catch(() => {})
    },
    addUser() {
      this.$refs.addFormRef.validate(async valid => {
        if (valid) {
          const { data: res } = await this.$http.post('/users', this.addForm)
          if (res.meta.status !== 201) {
            return this.$message.error(res.meta.msg)
          }
          this.$message.success(res.meta.msg)
          this.addDialogVisible = false
          this.getUserInfos()
        }
      })
    },
    addDialogClose() {
      this.$refs.addFormRef.resetFields()
    },
    async changeState(uid, state) {
      const { data: res } = await this.$http.put(
        'users/' + uid + '/state/' + state
      )
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.$message.success(res.meta.msg)
    },
    handelSizeChange(arg) {
      this.querycdt.pagesize = arg
      this.getUserInfos()
    },
    handelCurrentChange(arg) {
      this.querycdt.pagenum = arg
      this.getUserInfos()
    },
    async getUserInfos() {
      const { data: res } = await this.$http.get('/users', {
        params: this.querycdt
      })
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.querycdt.total = res.data.total
      this.userInfo = res.data.users
    }
  },
  data() {
    var checkMobile = (rule, value, callback) => {
      if (!/^1[2356789]\d{9}$/.test(value)) {
        return callback(new Error('手机号格式不正确!'))
      }
      callback()
    }
    return {
      editForm: {
        username: '',
        email: '',
        mobile: ''
      },
      editDialogVisible: false,
      editFormRules: {
        mobile: [
          { required: true, message: '请输入手机号', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },

      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      addDialogVisible: false,
      addFormRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' }
        ],
        password: [{ required: true, message: '请输入密码', trigger: 'blur' }],
        mobile: [
          { required: true, message: '请输入手机号', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },

      keywords: '',
      userInfo: [],
      querycdt: {
        query: '',
        pagenum: 1,
        pagesize: 2,
        total: 0
      }
    }
  }
}
</script>

<style lang="less" scoped>
</style>
