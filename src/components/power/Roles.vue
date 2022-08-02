<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card class="box-card">
      <el-button type="primary">添加角色</el-button>
      <el-table :data="RolesList" stripe border style="width: 100%">
        <el-table-column type="expand" width="80">
          <template slot-scope="scope">
            <!-- {{ scope.row }}   -->
            <el-row
              :class="['bdbottom', i1 === 0 ? 'bdtop' : '']"
              v-for="(items1, i1) in scope.row.children"
              :key="items1.id"
            >
              <el-col :span="5">
                <el-tag closable>
                  {{ items1.authName }}
                </el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <el-col :span="19">
                <el-row
                  v-for="(items2, i2) in items1.children"
                  :key="items2.id"
                  :class="[i2 == 0 ? '' : 'bdtop']"
                >
                  <el-col :span="6">
                    <el-tag type="success" closable>
                      {{ items2.authName }}
                    </el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="13">
                    <el-row>
                      <el-col>
                        <el-tag
                          type="warning"
                          v-for="(items3) in items2.children"
                          :key="items3.id"
                          closable
                          @close="removerRightById(scope.row, items3.id)"
                        >
                          {{ items3.authName }}
                        </el-tag>
                      </el-col>
                    </el-row>
                  </el-col>
                </el-row>
                <!-- <pre>{{items1.children}}</pre> -->
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index" label="#" width="80"> </el-table-column>
        <el-table-column prop="roleName" label="角色名称" width="400">
        </el-table-column>
        <el-table-column prop="roleDesc" label="角色描述" width="400">
        </el-table-column>
        <el-table-column prop="address" label="操作">
          <template slot-scope="scope">
            <el-button type="primary" icon="el-icon-edit">编辑</el-button>
            <el-button type="danger" icon="el-icon-delete">删除</el-button>
            <el-tooltip
              effect="dark"
              content="分配权限"
              placement="top"
              :enterable="false"
            >
              <el-button
                type="warning"
                icon="el-icon-setting"
                @click="showSetRightDialog(scope.row)"
                >分配权限</el-button
              >
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <el-dialog
      title="分配权限"
      :visible.sync="setRightDialogVisible"
      width="30%"
      @close="setRightDialogClose"
    >
      <el-tree
        :data="rightList"
        :props="treeProps"
        show-checkbox
        default-expand-all
        node-key="id"
        @node-click="handleNodeClick"
        :default-checked-keys="defKeys"
        ref="treeRef"
      ></el-tree>
      <span>
        <!-- <pre>{{ rightList }}</pre> -->
      </span>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      RolesList: [],
      setRightDialogVisible: false,
      rightList: [],
      treeProps: {
        children: "children",
        label: "authName",
      },
      defKeys: [],
      roleId: [],
    };
  },
  created() {
    this.getRolesList();
  },
  methods: {
    async getRolesList() {
      const { data: res } = await this.$http.get("roles");
      if (res.meta.status !== 200)
        return this.$message.error("获取角色列表失败");
      // console.log(res.data);
      this.RolesList = res.data;
      this.$message.success(res.meta.msg);
    },
    async removerRightById(role, rightId) {
      const confirmResult = await this.$confirm(
        "此操作将永久删除该文件, 是否继续?",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }
      ).catch((err) => err);
      if (confirmResult !== "confirm") {
        return this.$message.info("取消了删除！");
      }
      //   console.log('queren');
      const { data: res } = await this.$http.delete(
        `roles/${role.id}/rights/${rightId}`
      );
      if (res.meta.status !== 200) return this.$message.error("删除失败");
      this.$message.success(res.meta.msg);
      // this.getRolesList();
      role.children = res.data;
    },
    async showSetRightDialog(role) {
      this.roleId = role.id;
      const { data: res } = await this.$http.get("rights/tree");
      if (res.meta.status !== 200)
        return this.$message.error("获取权限列表失败");
      this.$message.success(res.meta.msg);
      console.log(res);
      this.rightList = res.data;
      this.getLeafList(role, this.defKeys);
      // console.log(role);
      this.setRightDialogVisible = true;
    },
    // 递归
    getLeafList(node, arr) {
      if (!node.children) return arr.push(node.id);
      node.children.forEach((items) => this.getLeafList(items, arr));
    },
    setRightDialogClose() {
      this.defKeys = [];
    },
    async allotRights() {
      const keys = [
        ...this.$refs.treeRef.getHalfCheckedKeys(),
        ...this.$refs.treeRef.getCheckedKeys(),
      ];
      const idStr = keys.join(",");
      // console.log(idStr);
      const { data: res } = await this.$http.post(
        `roles/${this.roleId}/rights`,
        { rids: idStr }
      );
      if (res.meta.status !== 200) return this.$message.error("角色授权失败");
      this.getRolesList()
      this.setRightDialogVisible = false
      this.$message.success(res.meta.msg);
    },
  },
};
</script>

<style lang='less' scoped>
.box-card {
  margin-top: 20px;
}
.el-tag {
  margin-top: 20px;
  margin-left: 20px;
}
.bdtop {
  border-top: 1px solid #eee;
  margin-top: 10px;
}
.bdbottom {
  border-bottom: 1px solid #eee;
  margin-bottom: 10px;
}
.bdbottom::after {
  margin-bottom: 6px;
}
.el-icon-caret-right {
  margin-left: 20px;
}
</style>