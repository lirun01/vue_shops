<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card class="box-card">
      <el-alert
        title="注意:只允许为三级分类设置相关参数"
        type="warning"
        show-icon
        :closable="false"
      >
      </el-alert>
      <el-row class="rowbox">
        <el-col>
          <span>选择商品分类</span>
          <!-- 级联选择框 -->
          <el-cascader
            expand-trigger="hover"
            v-model="selectCateKeys"
            :props="cateProps"
            :options="catelist"
            @change="handleChange"
            class="jilianbox"
          ></el-cascader>
          <!-- options指定数据源   -->
          <!-- props配置级联选择框 -->
        </el-col>
      </el-row>

      <el-tabs v-model="activeName" @tab-click="handleTabClick">
        <el-tab-pane label="动态参数" name="many">
          <el-button
            type="primary"
            :disabled="isBtnDisabled"
            @click="addmanyTableData"
            >添加参数</el-button
          >
          <el-table :data="manyTableData" border style="width: 100%">
            <el-table-column width="80" type="expand" Scoped slot>
              <template slot-scope="scope">
                <el-tag
                  v-for="(item, i) in scope.row.attr_vals"
                  :key="i"
                  class="tagbox"
                  closable
                  @close="handleClose(i, scope.row)"
                  >{{ item }}</el-tag
                >
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                >
                </el-input>
                <el-button
                  v-else
                  class="button-new-tag"
                  size="small"
                  @click="showInput(scope.row)"
                  >+ New Tag</el-button
                >
              </template>
            </el-table-column>
            <el-table-column type="index" label="#" width="80">
            </el-table-column>
            <el-table-column prop="attr_name" label="参数名称" width="600">
            </el-table-column>
            <el-table-column prop="address" label="操作">
              <template slot-scope="scope">
                <!-- {{ scope.row.attr_id }} -->
                <el-button
                  type="primary"
                  icon="el-icon-edit"
                  @click="showEditDialog(scope.row.attr_id)"
                  >修改</el-button
                >
                <el-button
                  type="danger"
                  icon="el-icon-delete"
                  @click="deletEditDialog(scope.row.attr_id)"
                  >删除</el-button
                >
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <el-tab-pane label="静态属性" name="only">
          <el-button
            type="primary"
            :disabled="isBtnDisabled"
            @click="addmanyTableData"
            >添加属性</el-button
          >
          <el-table :data="onlyTableData" border style="width: 100%">
            <el-table-column width="80" type="expand" Scoped slot>
              <template slot-scope="scope">
                <el-tag
                  v-for="(item, i) in scope.row.attr_vals"
                  :key="i"
                  class="tagbox"
                  closable
                  @close="handleClose(i, scope.row)"
                  >{{ item }}</el-tag
                >
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                >
                </el-input>
                <el-button
                  v-else
                  class="button-new-tag"
                  size="small"
                  @click="showInput(scope.row)"
                  >+ New Tag</el-button
                >
              </template>
            </el-table-column>
            <el-table-column type="index" label="#" width="80">
            </el-table-column>
            <el-table-column prop="attr_name" label="参数名称" width="600">
            </el-table-column>
            <el-table-column prop="address" label="操作">
              <template slot-scope="scope">
                <el-button
                  type="primary"
                  icon="el-icon-edit"
                  @click="showEditDialog(scope.row.attr_id)"
                  >修改</el-button
                >
                <el-button
                  type="danger"
                  icon="el-icon-delete"
                  @click="deletEditDialog(scope.row.attr_id)"
                  >删除</el-button
                >
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>

    <el-dialog
      :title="'添加' + titleText + '参数'"
      :visible.sync="catedialogVisible"
      width="30%"
      @close="addDialogClosed"
    >
      <el-form
        ref="addformRef"
        :rules="rules"
        :model="AddCateform"
        label-width="80px"
      >
        <el-form-item :label="titleText + '参数'" prop="attr_name">
          <el-input v-model="AddCateform.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="catedialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCateFormDT">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 修改 -->
    <el-dialog
      :title="'修改' + titleText + '参数'"
      :visible.sync="editdialogVisible"
      width="30%"
      @close="editDialogClosed"
    >
      <el-form
        ref="editformRef"
        :rules="rules"
        :model="editCateform"
        label-width="80px"
      >
        <el-form-item :label="titleText + '参数'" prop="attr_name">
          <el-input v-model="editCateform.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editdialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editCateFormDT">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      catelist: [],
      cateProps: {
        value: "cat_id",
        label: "cat_name",
        children: "children",
      },
      selectCateKeys: [],
      activeName: "many",
      onlyTableData: [],
      manyTableData: [],
      catedialogVisible: false,
      rules: {
        attr_name: [
          { required: true, message: "请输入动态参数", trigger: "blur" },
        ],
      },
      AddCateform: {
        attr_name: "",
      },
      editdialogVisible: false,
      editCateform: {
        attr_name: "",
      },
      editCateform: [],
    };
  },
  created() {
    this.getCateList();
  },
  methods: {
    async getCateList() {
      const { data: res } = await this.$http.get("categories");
      if (res.meta.status !== 200)
        return this.$message.error("获取商品数据失败");
      this.catelist = res.data;
      //   console.log(this.catelist);
      this.$message.success(res.meta.msg);
    },
    handleChange() {
      console.log(this.selectCateKeys);
      this.getParamsData();
    },
    handleTabClick() {
      this.getParamsData();
    },
    async getParamsData() {
      if (this.selectCateKeys.length !== 3) {
        this.selectCateKeys = [];
        this.manyTableData = [];
        this.onlyTableData = [];
        return;
      } else {
        const { data: res } = await this.$http.get(
          `categories/${this.caseId}/attributes`,
          {
            params: { sel: this.activeName },
          }
        );
        if (res.meta.status !== 200) {
          return this.$message.error("参数列表获取失败");
        }

        res.data.forEach((items) => {
          items.attr_vals = items.attr_vals ? items.attr_vals.split(" ") : [];
          items.inputVisible = false;
          items.inputValue = "";
          console.log(items.attr_vals);
        });

        console.log(res.data);
        if (this.activeName === "only") {
          // 静态
          this.onlyTableData = res.data;
        } else {
          // 动态
          this.manyTableData = res.data;
        }
        // console.log(this.onlyTableData);
        // console.log(this.manyTableData);
      }
    },
    addmanyTableData() {
      this.catedialogVisible = true;
    },
    addDialogClosed() {
      this.$refs.addformRef.resetFields();
    },
    addCateFormDT() {
      this.$refs.addformRef.validate(async (valid) => {
        if (!valid) return;
        const { data: res } = await this.$http.post(
          `categories/${this.caseId}/attributes`,
          {
            attr_name: this.AddCateform.attr_name,
            attr_sel: this.activeName,
          }
        );
        if (res.meta.status !== 201) return this.$message.error("创建失败");
        this.getParamsData();
        this.$message.success("添加参数成功");
        this.catedialogVisible = false;
      });
    },
    async showEditDialog(id) {
      const { data: res } = await this.$http.get(
        `categories/${this.caseId}/attributes/${id}`
      );
      if (res.meta.status !== 200) return this.$message.error("新参数获取失败");

      this.$message.success("获取新参数成功");
      console.log(res.data);
      this.editCateform = res.data;
      this.editdialogVisible = true;
    },
    editDialogClosed() {
      this.$refs.editformRef.resetFields();
    },
    editCateFormDT() {
      this.$refs.editformRef.validate(async (valid) => {
        if (!valid) return;
        const { data: res } = await this.$http.put(
          `categories/${this.caseId}/attributes/${this.editCateform.attr_id}`,
          {
            attr_name: this.editCateform.attr_name,
            attr_sel: this.activeName,
          }
        );
        if (res.meta.status !== 200) return this.$message.error("更新失败");
        this.getParamsData();
        this.editdialogVisible = false;
        this.$message.success("更新成功");
      });
    },
    async deletEditDialog(id) {
      const { data: res } = await this.$http.delete(
        `categories/${this.caseId}/attributes/${id}`
      );
      if (res.meta.status !== 200) return this.$message.error("删除失败");
      this.getParamsData();
      this.$message.success("删除成功");
    },
    handleInputConfirm(row) {
      if (row.inputValue.trim().length === 0) {
        row.inputValue = "";
        row.inputVisible = false;
        return;
      } else {
        row.attr_vals.push(row.inputValue.trim());
        row.inputValue = "";
        row.inputVisible = false;
        this.saveAttrVals(row);
      }
    },
    async saveAttrVals(row) {
      const { data: res } = await this.$http.put(
        `categories/${this.caseId}/attributes/${row.attr_id}`,
        {
          attr_name: row.attr_name,
          attr_sel: row.attr_sel,
          attr_vals: row.attr_vals.join(" "),
        }
      );
      if (res.meta.status !== 200) return this.$message.error("修改参数失败");
      //    this.saveAttrVals(row)
      this.$message.success("修改参数成功");
    },
    showInput(row) {
      row.inputVisible = true;
      this.$nextTick((_) => {
        this.$refs.saveTagInput.$refs.input.focus();
      });
    },
    handleClose(i, row) {
      row.attr_vals.splice(i, 1);
      this.saveAttrVals(row);
    },
  },
  computed: {
    isBtnDisabled() {
      if (this.selectCateKeys.length !== 3) {
        return true;
      } else {
        return false;
      }
    },
    caseId() {
      if (this.selectCateKeys.length === 3) {
        return this.selectCateKeys[2];
      } else {
        return null;
      }
    },
    titleText() {
      if (this.activeName === "many") {
        return "动态";
      } else {
        return "静态";
      }
    },
  },
};
</script>

<style lang='less' scoped>
.box-card {
  margin-top: 20px;
}
.rowbox {
  margin-top: 20px;
}
.jilianbox {
  margin-left: 20px;
}
.tagbox {
  margin-left: 20px;
}
.input-new-tag {
  margin-left: 20px;
  width: 120px;
}
.button-new-tag {
  margin-left: 20px;
}
</style>