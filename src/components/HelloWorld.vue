<template>
  <div class="hello">
    <button @click="dialogVisible = true">设置管家婆cookie</button>
    <button @click="wlndialogVisible = true">设置万里牛cookie</button>
    <div class="query-bar">
      <span>条形码</span>
      <el-input
        placeholder="请输入内容"
        v-model="productType"
        :disabled="false"
        size="small"
        class="tiaoxingma"
      ></el-input>
      <button @click="getList()">查询</button>
    </div>

    <div class="kou">
      <el-table :data="tableData" style="width: 100%">
        <el-table-column prop="name" label="商品名称" width="100"></el-table-column>
        <el-table-column prop="productType" label="商品条形码" width="100"></el-table-column>
        <el-table-column label="管家婆信息" align="center">
          <el-table-column prop="gjpPrice" label="数量"></el-table-column>
          <el-table-column prop="gjpPrice" label="成本均价"></el-table-column>
          <el-table-column prop="pid" label="仓库库存信息" width="110">
            <template slot-scope="scope">
              <el-button size="mini" @click="gjpSeeInfo(scope.$index, scope.row)"  :disabled="scope.row.pid == null">查看</el-button>
            </template>
          </el-table-column>
        </el-table-column>
        <el-table-column label="万里牛信息" align="center">
          <el-table-column prop="wlnSuof" label="锁定数量"></el-table-column>
          <el-table-column prop="wlnShij" label="实际数量"></el-table-column>
          <el-table-column prop="wlnKey" label="可用数量"></el-table-column>
          <el-table-column prop="wlnZait" label="在途数量"></el-table-column>
          <el-table-column prop="itemID" label="仓库库存数量" width="110">
            <template slot-scope="scope">
              <el-button size="mini" @click="wlnSeeInfo(scope.$index, scope.row)" :disabled="scope.row.itemID == null">查看</el-button>
            </template>
          </el-table-column>
        </el-table-column>
        <el-table-column prop="address" label="商品总数"></el-table-column>
        <el-table-column prop="tariff" label="关税费"></el-table-column>
      </el-table>
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page.sync="currentPage1"
        :page-size="50"
        layout="total, prev, pager, next"
        :total="total"
      ></el-pagination>
      <el-dialog title="管家婆" :visible.sync="dialogVisible" width="30%" :before-close="handleClose">
        <span slot="footer" class="dialog-footer">
          <el-form ref="gjpform" :model="gjpform" label-width="80px">
            <el-form-item label="内容">
              <el-input type="textarea" v-model="gjpform.cook"></el-input>
            </el-form-item>
          </el-form>
          <el-button @click="dialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="gjpSure()">确 定</el-button>
        </span>
      </el-dialog>
      <el-dialog
        title="万里牛"
        :visible.sync="wlndialogVisible"
        width="30%"
        :before-close="handleClose"
      >
        <span slot="footer" class="dialog-footer">
          <el-form ref="wlnform" :model="wlnform" label-width="80px">
            <el-form-item label="内容">
              <el-input type="textarea" v-model="wlnform.cook"></el-input>
            </el-form-item>
          </el-form>
          <el-button @click="wlndialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="wlnSure()">确 定</el-button>
        </span>
      </el-dialog>
      <el-dialog title="管家婆查看" :visible.sync="gjpListDialoge">
        <el-table :data="gjpData">
          <el-table-column property="id" label="行号" width="80"></el-table-column>
          <el-table-column property="name" label="仓库名称" width="150"></el-table-column>
          <el-table-column property="number" label="仓库数量" width="200"></el-table-column>
          <el-table-column property="price" label="仓库金额"  width="200"></el-table-column>
        </el-table>
      </el-dialog>
      <el-dialog title="万里牛查看" :visible.sync="wlnListDialoge">
        <el-table :data="wlnData">
          <el-table-column property="name" label="仓库名称" width="80"></el-table-column>
          <el-table-column property="wlnShij" label="实际" width="150"></el-table-column>
          <el-table-column property="wlnKey" label="可用" width="200"></el-table-column>
          <el-table-column property="wlnZait" label="在途"  width="200"></el-table-column>
        </el-table>
      </el-dialog>
    </div>
  </div>
</template>

<script>
export default {
  name: "HelloWorld",
  data() {
    return {
      tableData: [],
      gjpData: [],
      wlnData:[],
      dialogVisible: false, //管家婆对话框
      wlndialogVisible: false, // 万里牛对话框
      gjpListDialoge: false,  // 查看管家婆
      wlnListDialoge: false,  // 查看管家婆
      dialogFormVisible: false,
      form: {
        name: "",
        region: "",
        date1: "",
        date2: "",
        delivery: false,
        type: [],
        resource: "",
        desc: ""
      },
      gjpform: {
        cook: ""
      },
      wlnform: {
        cook: ""
      },
      productType: "",
      formLabelWidth: "120px",
      pageNo: 1,
      pageSize: 50,
      currentPage1: 1,
      total: 0,
    };
  },
  created() {
    this.getList();
  },
  methods: {
     gjpSeeInfo(index, row) {
        console.log(index, row);
        this.gjpListDialoge = true;
        let url = "http://112.74.57.236:1000/abutment/gjpProductC";
        this.axios.get(url+''+'?pid='+row.pid).then(res => {
            // debugger
          if (res.data.code == 200) {
            this.gjpData = res.data.data
            this.$message.success("请求查看成功！");
          } else {
            this.$message.error(res.data.msg);
          }
        });
      },
     wlnSeeInfo(index, row) {
        console.log(index, row);
        this.wlnListDialoge = true;
        let url = "http://112.74.57.236:1000/abutment/wlnProductC";
        let params = {
          itemID: row.itemID
        };
        this.axios.get(url+''+'?itemID='+row.itemID).then(res => {
          if (res.data.code == 200) {
            this.wlnData = res.data.data
            this.$message.success("请求查看成功！");
          } else {
            this.$message.error(res.data.msg);
          }
        });
      },
    handleSizeChange(val) {
      console.log(`每页 ${val} 条`);
      this.pageSize = val;
      this.getList();
    },
    handleCurrentChange(val) {
      console.log(`当前页: ${val}`);
      this.pageNo = val;
      this.getList();
    },
    gjpSure() {
      let url = "http://112.74.57.236:1000/abutment/setGjpCookie";
      let params = {
        cookie: this.gjpform.cook
      };
      this.axios.post(url, params).then(res => {
        if (res.data.code == 200) {
          this.dialogVisible = false
          this.$message.success("设置成功！");
        } else {
          this.$message.error(res.data.msg);
        }
      });
    },
    wlnSure() {
      let url = "http://112.74.57.236:1000/abutment/setWlnCookie";
      let params = {
        cookie: this.wlnform.cook
      };
      this.axios.post(url, params).then(res => {
        if (res.data.code == 200) {
          this.wlndialogVisible = false
          this.$message.success("设置成功！");
        } else {
          this.$message.error(res.data.msg);
        }
      });
    },
    handleClose(done) {
      this.$confirm("确认关闭？")
        .then(_ => {
          done();
        })
        .catch(_ => {});
    },
    getList() {
      let url = "http://112.74.57.236:1000/abutment/getProducts";
      let params = {
        pageNo: this.pageNo,
        pageSize: this.pageSize,
        productType: this.productType,
        start: 0
      };
      this.axios.post(url, params).then(res => {
        if (res.data.code == 200) {
          this.tableData = res.data.data;
          this.total = res.data.total;
          console.log(this.tableData);
          // debugger
          // sessionStorage.setItem('keyGen', JSON.stringify(res.data.data))
          this.$message.success("登录成功！");
        } else {
          this.$message.error(res.data.msg);
        }
      });
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.kou {
  width: 1100px;
  height: 1000px;
  background-color: red;
  background-color: green;
  border: brown solid 1px;
}

.query-bar {
  display: flex;
  flex-direction: row;
  align-items: center;
}

.tiaoxingma {
  display: inline-block;
  width: 50%;
}
/* .hello{
    display: flex;
    flex-direction: row;
    background-color:green;

  }
  .left{
    width: 180px;
    height: 1000px;
    background-color: blue;

  }
  .left li{
    list-style: none;
    height: 30px;
    line-height: 30px;
    border: solid 1px blueviolet;
  }
  .right{
    width: 1330px;
    height: 1000px;
    background-color: red;
  } */
</style>
