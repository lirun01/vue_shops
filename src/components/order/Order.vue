<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card class="box-card">
      <el-row>
        <el-col :span="8">
          <el-input
            placeholder="请输入内容"
            v-model="input3"
            class="input-with-select"
          >
            <el-button slot="append" icon="el-icon-search"></el-button>
          </el-input>
        </el-col>
      </el-row>
      <el-table :data="userInfoList" border style="width: 100%" class="lb">
        <el-table-column type="index" label="#" width="80"> </el-table-column>
        <el-table-column prop="order_number" label="订单编号" width="300">
        </el-table-column>
        <el-table-column prop="order_price" label="订单价格" width="180">
        </el-table-column>
        <el-table-column prop="pay_status" label="是否付款" width="200">
          <template slot-scope="scope">
            <el-tag effect="plain" v-if="scope.row.pay_status === '1'"
              >已付款</el-tag
            >
            <el-tag effect="danger" v-else>未付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="is_send" label="是否发货" width="200">
        </el-table-column>
        <el-table-column prop="create_time" label="下单时间" width="300">
          <template slot-scope="scope">
            {{ scope.row.create_time | dateFormat }}
          </template>
        </el-table-column>
        <el-table-column prop="address" label="操作">
          <template>
            <el-button
              type="primary"
              icon="el-icon-edit"
              circle
              @click="showBox"
            ></el-button>
            <el-button
              type="success"
              icon="el-icon-location"
              circle
              @click="showprojectBox"
            ></el-button>
          </template>
        </el-table-column>
      </el-table>

      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 10, 15, 20]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
        class="fy"
      >
      </el-pagination>
    </el-card>

    <el-dialog
      title="修改地址"
      :visible.sync="OrederdialogVisible"
      width="50%"
      @close="addressDialogClosed"
    >
      <el-form
        :rules="addressFormRules"
        ref="addressFormRef"
        label-width="100px"
        class="demo-ruleForm"
        :model="addressForm"
      >
        <el-form-item label="省市区/县" prop="address1">
          <el-cascader
            :options="cityData"
            :props="{ expandTrigger: 'hover' }"
            @change="handleChange"
            v-model="addressForm.address1"
          ></el-cascader>
        </el-form-item>

        <el-form-item label="详细地址" prop="address2">
          <el-input v-model="addressForm.address2"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="OrederdialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="OrederdialogVisible = false"
          >确 定</el-button
        >
      </span>
    </el-dialog>

    <!-- 物流信息 -->
    <el-dialog
      title="物流进度"
      :visible.sync="projectdialogVisible"
      width="50%"
    >
      <span>这是一段信息</span>
    </el-dialog>
  </div>
</template>

<script>
import cityData from "./citydata.js";
export default {
  data() {
    return {
      queryInfo: {
        query: "",
        pagenum: 1,
        pagesize: 10,
        user_id: "",
        pay_status: "",
      },
      total: "",
      userInfoList: [],
      OrederdialogVisible: false,
      addressForm: {
        address1: [],
        address2: "",
      },
      addressFormRules: {
        address1: [{ required: true, message: "请输入地址", trigger: "blur" }],
        address2: [
          { required: true, message: "请输入详细地址", trigger: "blur" },
        ],
      },
      cityData,
      projectdialogVisible:false
    };
  },
  created() {
    this.getOrderList();
  },
  methods: {
    async getOrderList() {
      const { data: res } = await this.$http.get("orders", {
        params: this.queryInfo,
      });
      if (res.meta.status !== 200) return this.$message.error("获取失败");
      this.userInfoList = res.data.goods;
      this.total = res.data.total;
      console.log(res.data);
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize;
      this.getOrderList();
    },
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage;
      this.getOrderList();
    },
    showBox() {
      this.OrederdialogVisible = true;
    },
    addressDialogClosed() {
      this.$refs.addressFormRef.resetFields();
    },
    async showprojectBox(){
      this.projectdialogVisible = true
      const {data:res} =await this.$http.get('/kuaidi/1106975712662')
      if (res.meta.status !==200) return this.$message.error('失败')
      console.log(res);
    }
  },
};
</script>

<style lang='less' scoped>
.box-card {
  margin-top: 20px;
}
.stepbox {
  margin-top: 20px;
}
.lb {
  margin-top: 20px;
}
.fy {
  margin-top: 20px;
}
.el-cascader {
  width: 100%;
}
</style>