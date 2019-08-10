<template>
  <div style="width: 100%;">
    <el-card style="margin-top: 20px;">
      <!-- <el-upload
        class="upload-demo"
        action="https://jsonplaceholder.typicode.com/posts/"
        @change="importf(this)">
        <el-button size="small" type="primary">点击上传</el-button>
        <span slot="tip" class="el-upload__tip">只能上传xlsx，xls格式文件</span>
      </el-upload> -->
      <input type="file" @change="importf(this)"/>
    </el-card>
    <el-card style="margin-top: 20px;">
      <el-button size="small" type="success" >一级降级</el-button>
      <el-button size="small" type="info" >二级降级</el-button>
      <el-button size="small" type="warning" >三级降级</el-button>
      <el-table :data="tableData" style="width: 100%">
        <el-table-column prop="name" label="名称" width="60">
        </el-table-column>
        <el-table-column prop="kindNum" label="型号" width="60">
        </el-table-column>
        <el-table-column prop="powerPressure" label="电源电压" width="80">
        </el-table-column>
        <el-table-column prop="enterPressure" label="输入电压" width="80">
        </el-table-column>
        <el-table-column prop="fanxiangV" label="反向电压" width="80">
        </el-table-column>
        <el-table-column prop="chaEnterOut" label="输入输出电压差" width="120">
        </el-table-column>
        <el-table-column prop="gongLv" label="功率" width="70">
        </el-table-column>
         <el-table-column prop="bigTempeture" label="最高结温" width="80">
        </el-table-column>
         <el-table-column prop="pinLv" label="频率" width="70">
        </el-table-column>
        <el-table-column prop="outFlut" label="输出电流" width="80">
        </el-table-column>
        <el-table-column prop="houmoMidu" label="厚膜功率密度" width="110">
        </el-table-column>
        <el-table-column prop="bomoMidu" label="薄膜功率密度" width="110">
        </el-table-column>
        <el-table-column prop="jifaV" label="集电极发射极电压" width="140">
        </el-table-column>
        <el-table-column prop="jiBigFlute" label="集电极最大电流" width="140">
        </el-table-column>

        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button
              size="mini"
              @click="handleEdit(scope.$index, scope.row)">调整降级
            </el-button>
            <el-button
              size="mini"
              type="danger"
              @click="handleDelete(scope.$index, scope.row)">删除
            </el-button>
          </template>
        </el-table-column>

      </el-table>

      <el-dialog title="用户信息" :visible.sync="dialogFormVisible">
        <el-form :model="form">
          <el-form-item label="名字" label-width="120px">
            <el-input v-model="form.name" auto-complete="off"></el-input>
          </el-form-item>
          <el-form-item label="出生日期" label-width="120px">
            <el-input v-model="form.date" auto-complete="off"></el-input>
          </el-form-item>
          <el-form-item label="出生地点" label-width="120px">
            <el-input v-model="form.address" auto-complete="off"></el-input>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="dialogFormVisible = false">取 消</el-button>
          <el-button type="primary" @click="modifyUser()">确 定</el-button>
        </div>
      </el-dialog>

    </el-card>
  </div>
</template>

<script>
import { requestUserQuery } from '@/api/user'
import xlsx from 'xlsx'
/* eslint-disable */
export default {
  name: 'PageTable',
  data () {
    return {
      tableData: [],
      rules: {},
      dialogFormVisible: false,
      form: {}
    }
  },
  mounted () {},
  methods: {
    importf (obj) {
      let _this = this
      let inputDom = this.$refs.inputer
      console.log(inputDom)
      this.file = event.currentTarget.files[0]
      var rABS = false
      var f = this.file
      var reader = new FileReader()
      FileReader.prototype.readAsBinaryString = function (f) {
        var binary = ""
        var rABS = false
        var wb
        var outdata
        var reader = new FileReader()
        reader.onload = function (e) {
          var bytes = new Uint8Array(reader.result)
          var length = bytes.byteLength
          for (var i = 0; i < length; i++) {
            binary += String.fromCharCode(bytes[i])
          }
          if (rABS) {
            wb = xlsx.read(btoa(fixdata(binary)), {
              type: 'base64'
            })
          } else {
            wb = xlsx.read(binary, {
              type: 'binary'
            })
          }

          outdata = xlsx.utils.sheet_to_json(wb.Sheets[wb.SheetNames[0]])

          let arr = []
          outdata.map(v => {
            let obj = {}
            obj.name = v.名称
            obj.kindNum = v.型号
            obj.powerPressure = v.电源电压
            arr.push(obj)
          })
          _this.accountList = [...arr]
          _this.tableData = _this.accountList
          console.log(_this.tableData)
        }
        reader.readAsArrayBuffer(f)
      }
      if (rABS) {
        reader.readAsArrayBuffer(f)
      } else {
        reader.readAsBinaryString(f)
      }
    },
    onSubmit (formName) {
      this.$refs[formName].validate((valid) => {
        if (valid) {
          requestUserQuery(this.formInline).then(res => {
            this.$message({
              message: '查询成功！',
              type: 'success'
            })
            this.pageTotal = res.data.length
            this.tableData = res.data
          })
        } else {
          console.log('error submit!!')
          return false
        }
      })
    },
    handleEdit (index, row) {
      this.form.index = index + (this.currentPage - 1) * this.pageSize
      this.form.name = row.name
      this.form.date = row.date
      this.form.address = row.address
      this.dialogFormVisible = true
    },
    handleDelete (index, row) {
      this.tableData.splice(index + (this.currentPage - 1) * this.pageSize, 1)
      this.pageTotal = this.tableData.length
      this.$message({
        message: '删除' + row.name + '成功！',
        type: 'success'
      })
    },
    handleSizeChange (size) {
      this.pagesize = size
    },
    handleCurrentChange (currentPage) {
      this.currentPage = currentPage
    },
    modifyUser () {
      this.tableData[this.form.index].name = this.form.name
      this.tableData[this.form.index].date = this.form.date
      this.tableData[this.form.index].address = this.form.address
      this.dialogFormVisible = false
      this.$message({
        message: '修改' + this.form.name + '成功！',
        type: 'success'
      })
    }
  }
}
</script>

<style scoped>
</style>
