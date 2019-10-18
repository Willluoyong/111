<template>
  <div class="hello">
    <button @click="gjpCookieDialoge = true">设置管家婆cookie</button>
    <button @click="wlnCookieDialoge = true">设置万里牛cookie</button>
    <br>
    <button @click="downloadFile">下载导出excel摸板</button>
    <button @click="uploadFile">上传excel</button>
    <div class="kou">
      <el-table :data="tableData" style="width: 100%">
        <el-table-column type="index" label="序号" width="50" ></el-table-column>
        <el-table-column prop="mailNo" label="流水号"  show-overflow-tooltip width="150"></el-table-column>
        <el-table-column prop="gjpCk" label="管家婆出货仓库名" width="100"></el-table-column>
        <el-table-column prop="createTime" label="上传时间" width="100"></el-table-column>
        <el-table-column prop="gjp" label="是否推送到管家婆" width="100"></el-table-column>
        <el-table-column prop="wln" label="是否推送到万里牛" width="100"></el-table-column>
        <el-table-column prop="productType" label="操作" width="380">
          <template slot-scope="scope">
            <el-button
              size="mini"
              @click="openGjpDialog(scope.$index, scope.row)"
              :disabled="scope.row.gjp == 1"
            >推送到管家婆</el-button>
            <el-button
              size="mini"
              @click="openWlnDialog(scope.$index, scope.row)"
              :disabled="scope.row.wln == 1"
            >推送到万里牛</el-button>
            <el-button
              size="mini"
              @click="seeDetail(scope.$index, scope.row)"
            >查看excel详情</el-button>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page.sync="currentPage1"
        :page-size="50"
        layout="total, prev, pager, next"
        :total="total"
      ></el-pagination>
      <el-dialog title="推送管家婆" :visible.sync="gjpdialogVisible" width="50%">
        <el-form ref="wlnform" :model="wlnform" label-width="80px">
            <el-form-item label="购买单位">
              <el-col :span="24">
                <el-select v-model="gjpform.dw" filterable placeholder="请选择购买单位" style="width: 100%;">
                  <el-option :label="item.fullname" :value="item.id+'&'+item.fullname" v-for="item in dwList" :key="item.id"></el-option>
                </el-select>
              </el-col>
            </el-form-item>
            <el-form-item label="经手人">
              <el-select v-model="gjpform.jsr" placeholder="请选择经手人" style="width: 100%;">
                <el-option :label="item.fullname" :value="item.id+'&'+item.fullname" v-for="item in jsrList" :key="item.id"></el-option>
              </el-select>
            </el-form-item>
            <el-form-item label="部门">
              <el-select v-model="gjpform.depart" placeholder="请选择部门" style="width: 100%;">
                <el-option :label="item.fullname" :value="item.id+'&'+item.fullname" v-for="item in departList" :key="item.id"></el-option>
              </el-select>
            </el-form-item>
          </el-form>
          <span slot="footer" class="dialog-footer">
            <el-button @click="gjpdialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="sendGjp()">确 定</el-button>
          </span>
      </el-dialog>
      <el-dialog
        title="推送万里牛"
        :visible.sync="wlndialogVisible"
        width="50%"
      >
          <el-form ref="wlnform" :model="wlnform" label-width="80px">
            <el-form-item label="订单号">
              <el-input  v-model="wlnform.bill_code"></el-input>
            </el-form-item>
            <el-form-item label="业务日期">
              <el-col :span="24">
                <el-date-picker type="date" placeholder="选择日期" v-model="wlnform.date" style="width: 100%;"></el-date-picker>
              </el-col>
            </el-form-item>
            <el-form-item label="供应商">
              <el-col :span="24">
                <el-select v-model="wlnform.supplier" filterable placeholder="请选择供应商" style="width: 100%;">
                  <el-option :label="item.unitName" :value="item.unitUid+'&'+item.unitName" v-for="item in gysList" :key="item.unitUid"></el-option>
                </el-select>
              </el-col>
            </el-form-item>
            <el-form-item label="仓库">
              <el-select v-model="wlnform.storage" placeholder="请选择仓库" style="width: 100%;">
                <el-option :label="item.storageName" :value="item.storageUid+'&'+item.storageName" v-for="item in ckList" :key="item.storageUid"></el-option>
              </el-select>
            </el-form-item>
            <el-form-item label="备注">
              <el-input  v-model="wlnform.remark"></el-input>
            </el-form-item>
            <el-form-item label="业务员">
             <el-select v-model="wlnform.person" placeholder="请选择业务员" style="width: 100%;">
               <el-option :label="item.operName" :value="item.operUid+'&'+item.operUid" v-for="item in personList" :key="item.operUid"></el-option>
              </el-select>
            </el-form-item>
          </el-form>
          <span slot="footer" class="dialog-footer">
            <el-button @click="wlndialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="sendWln()">确 定</el-button>
          </span>
      </el-dialog>
      <el-dialog title="查看详情" :visible.sync="detailDialog"   width="68%" center>
        <el-table :data="detailData">
          <el-table-column type="index" label="序号" width="50"></el-table-column>
          <el-table-column property="produceName" show-overflow-tooltip label="商品名称" width="80"></el-table-column>
          <el-table-column property="productType" show-overflow-tooltip label="商品条码" width="100"></el-table-column>
          <el-table-column property="quanality" label="出货数量" width="80"></el-table-column>
          <el-table-column property="price" label="出货单价" width="80"></el-table-column>
          <el-table-column property="money" label="出货总价" width="80"></el-table-column>
          <el-table-column property="gjpCk" label="管家婆出货仓库" width="120"></el-table-column>
          <el-table-column property="tariff" label="关税费" width="80"></el-table-column>
          <el-table-column property="outPrice" label="万里牛入库单价" width="80"></el-table-column>
          <el-table-column property="outMoney" label="万里牛入库总价" width="80"></el-table-column>
        </el-table>
      </el-dialog>
      <!-- <el-dialog title="万里牛查看" :visible.sync="wlnListDialoge">
        <el-table :data="wlnData">
          <el-table-column property="name" label="仓库名称" width="80"></el-table-column>
          <el-table-column property="wlnShij" label="实际" width="150"></el-table-column>
          <el-table-column property="wlnKey" label="可用" width="200"></el-table-column>
          <el-table-column property="wlnZait" label="在途" width="200"></el-table-column>
        </el-table>
      </el-dialog> -->
      <el-dialog title="管家婆" :visible.sync="gjpCookieDialoge" width="30%" >
          <el-form ref="gjpCookieForm" :model="gjpCookieForm" label-width="80px">
            <el-form-item label="内容">
              <el-input type="textarea" v-model="gjpCookieForm.cook"></el-input>
            </el-form-item>
          </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="gjpCookieDialoge = false">取 消</el-button>
          <el-button type="primary" @click="gjpSure()">确 定</el-button>
        </span>
      </el-dialog>
      <el-dialog
        title="万里牛"
        :visible.sync="wlnCookieDialoge"
        width="30%"
      >
          <el-form ref="wlnCookieForm" :model="wlnCookieForm" label-width="80px">
            <el-form-item label="内容">
              <el-input type="textarea" v-model="wlnCookieForm.cook"></el-input>
            </el-form-item>
          </el-form>
          <span slot="footer" class="dialog-footer">
            <el-button @click="wlnCookieDialoge = false">取 消</el-button>
            <el-button type="primary" @click="wlnSure()">确 定</el-button>
          </span>
      </el-dialog>
    </div>
  </div>
</template>

<script>
export default {
  name: "Second",
  data() {
    return {
      tableData: [],
      detailData: [],
      // wlnData: [],
      gjpCookieDialoge: false, //管家婆对话框
      wlnCookieDialoge: false, // 万里牛对话框
      dialogVisible: false, //管家婆对话框
      gjpdialogVisible: false, // 管家婆对话框
      wlndialogVisible: false, // 万里牛对话框
      detailDialog: false, // 查看管家婆
      wlnListDialoge: false, // 查看万里牛
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
        dw : "", //购买单位
        jsr : "", //
        depart : "", //
      },
      gjpCookieForm: {
        cook : "", 
      },
      wlnCookieForm: {
        cook : "", 
      },
      wlnform: {
        bill_code: "",
        date: "",
        supplier: "",
        storage: "",
        remark: "",
        person: "",
      },
      dwList:[], //单位list
      jsrList:[], //经手人list
      departList:[], //部门list
      gysList:[],
      ckList:[],
      personList:[],
      ckName:'', //仓库名
      ckId:'', // 仓库id
      mailNo:'',
      productType: "",
      formLabelWidth: "120px",
      pageNo: 1,
      pageSize: 50,
      currentPage1: 1,
      total: 0
    };
  },
  created() {
    this.getList();
  },
  methods: {
    seeDetail(index, row) {
      console.log(index, row);
      this.detailDialog = true;
      let url = "/getExcelInfos";
      this.axios.get(url + "" + "?mailNo=" + row.mailNo).then(res => {
        if (res.data.code == 200) {
          this.detailData = res.data.data;
          this.$message.success("请求查看成功！");
        } else {
          this.$message.error(res.data.msg);
        }
      });
    },
    openGjpDialog(index, row) {
      console.log(index, row);
      this.ckName = row.gjpCk;
      this.ckId = row.gjpCkId;
      this.mailNo = row.mailNo;
      this.gjpdialogVisible = true;
      let url = "/gjpDanWei";
      let params = {
          "pageNo": 1,
          "pageSize": 200,
          "productType": "",
          "start": 0
        }
      this.axios.post(url,params).then(res => {
        if (res.data.code == 200) {
          this.dwList = res.data.data;
        } else {
          this.$message.error(res.data.msg);
        }
      });
      let url2 = "/gjpJsr";
      let params2 = {
          "pageNo": 1,
          "pageSize": 200,
          "productType": "",
          "start": 0
        }
      this.axios.post(url2,params2).then(res => {
        if (res.data.code == 200) {
          this.jsrList = res.data.data;
        } else {
          this.$message.error(res.data.msg);
        }
      });
      let url3 = "/gjpBm";
      let params3 = {
          "pageNo": 1,
          "pageSize": 200,
          "productType": "",
          "start": 0
        }
      this.axios.post(url3,params3).then(res => {
        if (res.data.code == 200) {
          this.departList = res.data.data;
        } else {
          this.$message.error(res.data.msg);
        }
      });
    },
    openWlnDialog(index, row) {
      console.log(index, row);
      this.mailNo = row.mailNo;
      this.wlndialogVisible = true;
      let url = "/wlnGys";
      let params = {
          "pageNo": 1,
          "pageSize": 200,
          "productType": "",
          "start": 0
        }
      this.axios.post(url,params).then(res => {
        if (res.data.code == 200) {
          this.gysList = res.data.data;
        } else {
          this.$message.error(res.data.msg);
        }
      });
      let url2 = "/wlnCk";
      let params2 = {
          "pageNo": 1,
          "pageSize": 200,
          "productType": "",
          "start": 0
        }
      this.axios.post(url2,params2).then(res => {
        if (res.data.code == 200) {
          this.ckList = res.data.data;
        } else {
          this.$message.error(res.data.msg);
        }
      });
      let url3 = "/wlnYwy";
      let params3 = {
          "pageNo": 1,
          "pageSize": 200,
          "productType": "",
          "start": 0
        }
      this.axios.post(url3,params3).then(res => {
        if (res.data.code == 200) {
          this.personList = res.data.data;
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
      let url = "/setGjpCookie";
      let params = {
        cookie: this.gjpCookieForm.cook
      };
      this.axios.post(url, params).then(res => {
        if (res.data.code == 200) {
          this.gjpCookieDialoge = false
          this.$message.success("设置成功！");
        } else {
          this.$message.error(res.data.msg);
        }
      });
    },
    wlnSure() {
      let url = "/setWlnCookie";
      let params = {
        cookie: this.wlnCookieForm.cook
      };
      this.axios.post(url, params).then(res => {
        if (res.data.code == 200) {
          this.wlnCookieDialoge = false
          this.$message.success("设置成功！");
        } else {
          this.$message.error(res.data.msg);
        }
      });
    },
    sendGjp() {
      let url = "/sendExcelOutFromGjp";
      let request = {
          "bfullname": this.gjpform.dw.split('&')[1],
          "btypeid": this.gjpform.dw.split('&')[0],
          "efullname": this.gjpform.jsr.split('&')[1],
          "etypeid": this.gjpform.jsr.split('&')[0],
          "inputfullname": "",
          "inputno": "",
          "kfullname": this.ckName,
          "ktypeid": this.ckId,
          "mailNo": this.mailNo
      }
      this.axios.post(url, request).then(res => {
        if (res.data.code == 200) {
          this.wlndialogVisible = false;
          this.$message.success("操作成功！");
        } else {
          this.$message.error(res.data.msg);
        }
      });
    },
    sendWln() {
      let url = "/sendExcelOutFromWln";
      let request = {
          "bill_code": this.wlnform.bill_code,
          "createTime": this.wlnform.date,
          "mailNo": this.mailNo,
          "remark": this.wlnform.remark,
          "saleman": this.wlnform.person.split('&')[1],
          "salemanUid": this.wlnform.person.split('&')[0],
          "storage_name": this.wlnform.storage.split('&')[1],
          "storage_uid": this.wlnform.storage.split('&')[0],
          "supplier_name":this.wlnform.supplier.split('&')[1],
          "supplier_uid": this.wlnform.supplier.split('&')[0],
        }
      this.axios.post(url, request).then(res => {
        if (res.data.code == 200) {
          this.wlndialogVisible = false;
          this.$message.success("操作成功！");
        } else {
          this.$message.error(res.data.msg);
        }
      });
    },
    downloadFile() {
      let url = "http://112.74.57.236:1000/file/download";
      window.open(url);
    },
    uploadFile() {
    },
    handleClose(done) {
      this.$confirm("确认关闭？")
        .then(_ => {
          done();
        })
        .catch(_ => {});
    },
    getList() {
      let url = "/getExcelList";
      let params = {
        pageNo: this.pageNo,
        pageSize: this.pageSize,
        productType: '',
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

  .dywidth{
    width: 80% !important;
  }
</style>

