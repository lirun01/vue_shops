<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card class="box-card">
      <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
      <tree-table
        :data="catelist"
        :columns="columns"
        class="tabletop"
        :selection-type="false"
        :expand-type="false"
        show-index
      >
        <template slot="isok" slot-scope="scope">
          <i
            class="el-icon-success"
            v-if="scope.row.cat_deleted === false"
            style="color: lightgreen"
          ></i>
          <i class="el-icon-error" v-else style="color: red"></i>
          <!-- {{ scope.row.cat_level }} -->
        </template>

        <temlplate slot="order" slot-scope="scope">
          <el-tag v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag type="success" v-else-if="scope.row.cat_level === 1"
            >二级</el-tag
          >
          <el-tag type="warning" v-else>三级</el-tag>
        </temlplate>

        <temlplate slot="opt" slot-scope="scope">
          <el-button type="primary" icon="el-icon-edit">编辑</el-button>
          <el-button type="danger" icon="el-icon-delete">删除</el-button>
        </temlplate>
      </tree-table>

      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="currentPage4"
        :page-sizes="[2, 3, 5, 7]"
        :page-size="100"
        layout="total, sizes, prev, pager, next, jumper"
        :total="this.total"
        class="treeTable"
      >
      </el-pagination>

      <el-dialog
        title="添加分类"
        :visible.sync="adddialogVisible"
        width="30%"
        :before-close="handleClose"
        @close="addCatelist"
      >
        <el-form
          ref="addCateFormRef"
          :model="addCateForm"
          label-width="80px"
          :rules="rules"
        >
          <el-form-item label="分类名称" prop="cat_name">
            <el-input v-model="addCateForm.cat_name"></el-input>
          </el-form-item>
          <el-form-item label="父级分类">
            <el-cascader
              v-model="selectKeys"
              :options="prentCaseList"
              :props="cascderProps"
              @change="parentCateChanged"
              expand-trigger="hover"
              clearable
              change-on-select
            ></el-cascader>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="adddialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addCate">确 定</el-button>
        </span>
      </el-dialog>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 2,
      },
      catelist: [],
      total: "",
      columns: [
        {
          label: "分类名称",
          prop: "cat_name",
        },
        {
          label: "是否有效",
          type: "template",
          template: "isok",
        },
        {
          label: "排序",
          type: "template",
          template: "order",
        },
        {
          label: "操作",
          type: "template",
          template: "opt",
        },
      ],
      adddialogVisible: false,
      rules: {
        cat_name: [
          { required: true, message: "请输入活动名称", trigger: "blur" },
          { min: 3, max: 5, message: "长度在 3 到 5 个字符", trigger: "blur" },
        ],
      },
      addCateForm: {
        cat_name: "",
        cat_pid: 0,
        cat_level: 0,
      },
      prentCaseList: [],
      cascderProps: {
        value: "cat_id",
        label: "cat_name",
        children: "children",
      },
      selectKeys: [],
    };
  },
  created() {
    this.getCateList();
  },
  methods: {
    async getCateList() {
      const { data: res } = await this.$http.get("categories", {
        params: this.queryInfo,
      });
      if (res.meta.status !== 200)
        return this.$message.error("获取分类列表失败");
      this.catelist = res.data.result;
      this.total = res.data.total;
      // console.log(res.data.total);
      this.$message.success(res.meta.msg);
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize;
      this.getCateList();
    },
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage;
      this.getCateList();
    },
    showAddCateDialog() {
      this.getParentCateList();
      this.adddialogVisible = true;
    },
    async getParentCateList() {
      const { data: res } = await this.$http.get("categories", {
        params: { type: 2 },
      });
      // console.log(res);
      if (res.meta.status !== 200) return this.$message.error("获取失败");
      this.prentCaseList = res.data;
      // console.log(res.data);
      this.$message.success(res.meta.msg);
    },
    parentCateChanged() {
      // console.log(this.selectKeys);
      if (this.selectKeys.length > 0) {
        this.addCateForm.cat_pid = this.selectKeys[this.selectKeys.length - 1];
        this.addCateForm.cat_level = this.selectKeys.length;
      } else {
        this.addCateForm.cat_pid = 0;
        this.addCateForm.cat_level = 0;
      }
    },
    async addCate() {
      // console.log(this.addCateForm);
      const { data: res } = await this.$http.post(
        "categories",
        this.addCateForm
      );
      if (res.meta.status !== 201) return this.$message.error("创建失败");
      this.getCateList();
      this.$message.success(res.meta.msg);
      this.adddialogVisible = false;
    },
    addCatelist() {
      this.$refs.addCateFormRef.resetFields();
      this.selectKeys = []
      this.selectKeys.cat_pid = 0;
      this.selectKeys.cat_level = 0;
    },
  },
};
</script>

<style lang='less' scoped>
.box-card {
  margin-top: 20px;
}
.tabletop {
  margin-top: 20px;
}
.treeTable {
  margin-top: 20px;
}
</style>