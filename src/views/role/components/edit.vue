<template>
  <dialog-layout :visible.sync="visible" :title="title" :loading="loading" height="60vh">
    <div>
      <el-tabs v-model="active_tab">
        <el-tab-pane label="角色信息" name="role-info">
          <role-info
            :name.sync="info.name"
            :desc.sync="info.desc"
            :role-has-users="info.role_has_users"
          ></role-info>
        </el-tab-pane>
        <el-tab-pane label="功能权限" name="function-permission">
          <function-permission
            ref="function-permission"
            :permission-list="permission_list"
            :permission-ids.sync="info.permission_ids"
          ></function-permission>
        </el-tab-pane>
        <el-tab-pane label="数据权限" name="data-permission">
          <data-permission
            ref="data-permission"
            :department-list="department_list"
            :parent-game-list="parent_game_list"
            :data-permission.sync="info.data_permission"
          ></data-permission>
        </el-tab-pane>
      </el-tabs>
    </div>
  </dialog-layout>
</template>

<script>
import { dialog_common } from '@/layout/mixin/dialog_common'
import RoleInfo from './role-info'
import FunctionPermission from './function-permission'
import DataPermission from './data-permission'
import { getPermissions, roleInfo, addRole, updRole } from '@/api/role'
import { departmentUserTree } from '@/api/account'
import { getAllParentGame } from '@/api/common'
import { MyObject } from '@/utils/myObject'

export default {
  name: 'Edit',
  components: {
    RoleInfo,
    FunctionPermission,
    DataPermission
  },
  mixins: [dialog_common],
  data() {
    return {
      active_tab: 'role-info',
      info: {
        id: 0,
        name: '',
        desc: '',
        role_has_users: [],
        permission_ids: [],
        data_permission: {}
      },
      permission_list: [],
      parent_game_list: [],
      department_list: []
    }
  },
  computed: {
    title: function() {
      return this.id ? '编辑角色' : '新增角色'
    }
  },
  methods: {
    init_api() {
      if (this.id) {
        return roleInfo({ id: this.id })
      } else {
        return new Promise((resolve) => {
          resolve({ result: {}})
        })
      }
    },
    after_init() {
      if (this.permission_list.length === 0) {
        getPermissions().then((res) => {
          this.permission_list = res.result
        })
      }
      if (this.parent_game_list.length === 0) {
        getAllParentGame().then((res) => {
          console.log(res)
          this.parent_game_list = new MyObject(res.result).toArray()
        })
      }
      if (this.department_list.length === 0) {
        departmentUserTree().then((res) => {
          this.department_list = res.result
        })
      }
    },
    submit_api() {
      if (this.info.name === '') {
        this.$message.error('请输入角色名称')
        return new Promise((resolve, reject) => {
          reject()
        })
      }
      const role_info = Object.assign({}, this.info)
      delete role_info.role_has_users
      delete role_info.permissions
      if (role_info.id) {
        return updRole(role_info)
      } else {
        delete role_info.id
        return addRole(role_info)
      }
    },
    before_close() {
      this.$refs['function-permission'].$refs.tree.setCheckedKeys([])
      this.info.id = 0
      this.info.name = ''
      this.info.desc = ''
      this.info.role_has_users = []
      this.info.permission_ids = []
      this.info.data_permission = {}
    }
  }
}
</script>

<style scoped></style>
