<template>
  <section>
    <el-row>
      <el-col :span="24">
        <!--表单-->
        <el-form :inline="true" :model="formInline" class="demo-form-inline">
          <el-form-item label="商品名称">
            <el-input v-model="formInline.user.name" placeholder="Pine Green/Black/Sail" style="width: 140px;"></el-input>
          </el-form-item>
          <el-form-item label="提醒状态">
            <el-input v-model="formInline.user.alertStatus" placeholder="1" style="width: 140px;"></el-input>
          </el-form-item>
          <el-form-item label="年份">
            <el-date-picker
              v-model="formInline.user.date"
              align="right"
              type="year"
              placeholder="选择年份">
            </el-date-picker>
          </el-form-item>
          <el-form-item label="地址">
            <el-cascader expand-trigger="hover" :options="options" v-model="formInline.user.address"></el-cascader>
          </el-form-item>
          <el-form-item label="籍贯">
            <el-select v-model="formInline.user.place" placeholder="请选择">
              <el-option
                v-for="item in places"
                :label="item.label"
                :value="item.value">
              </el-option>
            </el-select>
          </el-form-item>
          <el-button type="primary" @click="onSubmit">查询</el-button>
          <a href="javascript:;" id="download" style="background-color:#409EFF;color: #fff;padding: 12px 10px!important;margin-left: 10px!important;border-radius:4px " @click="download()" download="download.csv">导出数据</a>
        </el-form>
        <!--表格-->
        <el-table
          :data="tableData"
          border
          style="width: 100%"
          :span-method="objectSpanMethod">
          <el-table-column type="selection">
          </el-table-column>
          <el-table-column>
            <template scope="scope">
              <img :src="scope.row.img" width="60" height="60" class="head_pic"/>
            </template>
          </el-table-column>
          <el-table-column
            prop="mid"
            label="商品编码"
            width="120">
          </el-table-column>
          <el-table-column
            prop="brandId"
            label="品牌id"
            width="100">
          </el-table-column>
          <el-table-column
            prop="brandName"
            label="品牌名称"
            width="100">
          </el-table-column>
          <el-table-column
            prop="mname"
            label="商品名"
            width="140">
          </el-table-column>
          <el-table-column
            prop="rmb"
            label="金额"
            width="100">
          </el-table-column>
          <el-table-column
            prop="inventoryStatus"
            label="状态">
          </el-table-column>
          <el-table-column
            label="提醒状态">
            <template slot-scope="scope">
              {{scope.row.isAlter==1?"已提醒":"无"}}
            </template>
          </el-table-column>
          <el-table-column
            prop="size"
            label="尺寸">
            <template slot-scope="scope">
              {{scope.row.saleSize}}
            </template>
          </el-table-column>
          <el-table-column
            prop="size"
            label="全部尺寸">
            <template slot-scope="scope">
              {{scope.row.allSize}}
            </template>
          </el-table-column>
          <el-table-column label="操作">
            <template scope="scope">
              <el-button type="primary" size="small" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
              <el-button type="danger" size="small" @click="handleDelete(scope.$index, scope.row)">删除</el-button>
            </template>
          </el-table-column>
        </el-table>
        <div class="block">
          <el-pagination
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
            :current-page="currentPage"
            :page-size="100"
            layout="prev, pager, next, jumper"
            :total="1000">
          </el-pagination>
        </div>
      </el-col>
    </el-row>
    <el-dialog title="修改提醒信息" :visible="dialogFormVisible" size="tiny">
      <el-form ref="goods" :model="goods" label-width="80px">
        <el-form-item label="提醒方式" prop="way">
          <el-checkbox-group v-model="checkedSize">
            <el-checkbox v-for="(item, index) in goods.allSize"  :key="index" :label="item" >{{item}}</el-checkbox>
          </el-checkbox-group>
        </el-form-item>

        <el-form-item label="是否流行" prop="type">
            <el-radio-group  v-model="goods.isPopular">
              <el-radio  :label="0">否</el-radio>
              <el-radio  :label="1">是</el-radio>
            </el-radio-group>
          </el-form-item>

          <el-form-item label="模式" prop="type">
            <el-radio-group  v-model="goods.mode">
              <el-radio  :label="1">库存模式</el-radio>
              <el-radio  :label="2">促销模式</el-radio>
            </el-radio-group>
          </el-form-item>

        <el-form-item label="地址">
          <el-input v-model="form.address"></el-input>
        </el-form-item>

        <el-form-item label="出生日期">
          <el-date-picker type="date" placeholder="选择日期" v-model="form.date" style="width: 100%;" ></el-date-picker>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="handleSave" :loading="editLoading">修改</el-button>
          <el-button @click="dialogFormVisible = false">取消</el-button>
        </el-form-item>
      </el-form>
    </el-dialog>
  </section>
</template>
<script type="text/ecmascript-6">
  const ERR_OK = "0";
  export default {
    data () {
      return {
        formInline: {
          user: {
            name: '',
            alertStatus: '',
            date: '',
            address: [],
            place: ''
          }
        },
        tableData: [],
        originTableData: [],
        queryTableData: [],
        options: [],
        places: [],
        dialogFormVisible: false,
        editLoading: false,
        form: {
          name: '',
          address: '',
          date: '',
        },
        goods: {
          brandId: '',
          brandName: '',
          id: '',
          img: '',
          inventoryStatus: '',
          mname: '',
          mid: '',
          rmb: '',
          size: '',
          mode: '',
          isPopular: '',
        },
        checkedSize: [],
        currentPage: 4,
        table_index: 999,
        shoeSize: [],
      };
    },
    created () {
      var body = {
        "appId": "mf",
        "language": "zh-CN",
        "requestId": "5026a2c8-cc84-4ef6-9a73-5f2901301ad3",
        "sign": "a68da1ce0c85b031206f720218e5cba9",
        "timeZone": "GMT+8",
        "timestamp": 1486975919,
        "orderId": 164397097
      };
      this.$http.post('http://106.54.189.47:8088/by/beyond/queryFavorites', body).then((response) => {
        response = response.data;
        if (response.errorCode === ERR_OK) {
          this.originTableData = this.tableData = response.data.goods;
        }
      });
      this.$http.get('/api/getOptions').then((response) => {
        response = response.data;
        if (response.code === ERR_OK) {
          this.options = response.datas;
          this.places = response.places;
        }
      });
    },
    methods: {
      onSubmit () {
        this.queryTableData = [];
        if (this.formInline.user.name === '' && this.formInline.user.alertStatus === ''){
          this.tableData = this.originTableData;
          return
        } else {
          for (var i = 0; i < this.tableData.length; i++) {
            var item = this.tableData[i];
            if (item.mname === this.formInline.user.name){
              this.queryTableData.push(item)
            } else if (item.isAlter + '' === this.formInline.user.alertStatus){
              this.queryTableData.push(item)
            }
          }
          this.tableData = this.queryTableData;
        }
        this.formInline.user.name = '';
        this.formInline.user.alertStatus = '';
        // this.$message('模拟数据，这个方法并不管用哦~');
      },
      handleDelete (index, row) {
        this.tableData.splice(index, 1);
        this.$message({
          message: "操作成功！",
          type: 'success'
        });
        console.log(row);
        this.$http.post('http://106.54.189.47:8088/by/beyond/deleteMp', row).then((response) => {
          response = response.data;
          if (response.data.err.errorCode === ERR_OK) {
            this.tableData = response.data.goods;
          }
        });
      },
      handleEdit (index, row) {
        this.dialogFormVisible = true;
        this.form = Object.assign({}, row);
        this.goods = Object.assign({}, row);
        this.table_index = index;
      },
      handleSave () {
        this.$confirm('确认提交吗？', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          cancelButtonClass: 'cancel'
        }).then(() => {
          this.editLoading = true;
          let date = this.form.date;
          console.log(this.goods);
          console.log(this.checkedSize);
          var update = this.goods;
          update.saleSize = this.checkedSize;
          console.log(update);
          this.$http.post('http://106.54.189.47:8088/by/beyond/alertMp', update).then((response) => {
            response = response.data;
            this.checkedSize = [];
            if (response.data.err.errorCode === ERR_OK) {
              this.tableData = response.data.goods;
            }
          });
          if (typeof date === "object") {
            date = [date.getFullYear(), (date.getMonth() + 1), (date.getDate())].join('-');
            this.form.date = date
          }
//          this.tableData[this.table_index] = this.form;
          this.tableData.splice(this.table_index, 1, this.form);
          this.$message({
            message: "操作成功！",
            type: 'success'
          });
          this.editLoading = false;
          this.dialogFormVisible = false;
        }).catch(() => {

        });
      },
      download: function() {
        console.log("xiazai")
        var obj = document.getElementById('download');
        var str = "姓名,出生日期,地址\n";
        for (var i = 0; i < this.tableData.length; i++) {
          var item = this.tableData[i];
          str += item.name + ',' + item.date + ',' + item.address;
          str += "\n";
        }
        console.log(obj)
        str = encodeURIComponent(str);
        console.log(str)
        obj.href = "data:text/csv;charset=utf-8,\ufeff" + str;
        obj.download = "download.csv";
      },
      handleSizeChange(val) {
        console.log(`每页 ${val} 条`);
      },
      handleCurrentChange(val) {
        this.currentPage = val;
        console.log(`当前页: ${val}`);
      },
      arraySpanMethod({ row, column, rowIndex, columnIndex }) {
        if (rowIndex % 2 === 0) {
          if (columnIndex === 0) {
            return [1, 2];
          } else if (columnIndex === 1) {
            return [0, 0];
          }
        }
      },
      objectSpanMethod({ row, column, rowIndex, columnIndex }) {
        if (columnIndex === 0) {
          if (rowIndex % 2 === 0) {
            return {
              rowspan: 2,
              colspan: 1
            };
          } else {
            return {
              rowspan: 0,
              colspan: 0
            };
          }
        }
      }
    }
  };
</script>
<style>
  .el-pagination {
    text-align: center;
    margin-top: 30px;
  }
  .el-message-box__btns .cancel {
    float: right;
    margin-left: 10px;
  }
</style>
