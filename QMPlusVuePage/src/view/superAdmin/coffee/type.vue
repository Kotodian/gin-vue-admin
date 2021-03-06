<template>
  <div>
    <div class="button-box clearflex">
      <el-button @click="addCoffeeType" type="primary">新增咖啡类型</el-button>
    </div>
    <el-table :data="tableData" border stripe>
      <el-table-column label="图片" min-width="50">
        <template slot-scope="scope">
          <div :style="{'textAlign':'center'}">
            <img :src="scope.row.image" height="50" width="50" />
          </div>
        </template>
      </el-table-column>
      <el-table-column label="咖啡类型名称" min-width="150" prop="name"></el-table-column>
      <el-table-column label="咖啡类型代码" min-width="150" prop="code"></el-table-column>
      <el-table-column fixed="right" label="操作" width="300">
        <template slot-scope="scope">
          <el-button @click="deleteCoffeeType(scope.row)" size="small" type="text">删除咖啡类型</el-button>
          <el-button @click="editCoffeeType(scope.row)" size="small" type="text">编辑咖啡类型</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      :current-page="page"
      :page-size="pageSize"
      :page-sizes="[10, 30, 50, 100]"
      :style="{float:'right',padding:'20px'}"
      :total="total"
      @current-change="handleCurrentChange"
      @size-change="handleSizeChange"
      layout="total, sizes, prev, pager, next, jumper"
    ></el-pagination>

    <el-dialog :visible.sync="addCoffeeTypeDialog" custom-class="user-dialog" :title="titleMap[dialogTitle]">
      <el-form :model="coffeeTypeInfo" :rules="rules" ref="coffeeTypeInfo">
        <el-form-item label="类型名称" label-width="80px" required>
          <el-input v-model="coffeeTypeInfo.name"></el-input>
        </el-form-item>
        <el-form-item label="类型代码" label-width="80px" required>
          <el-input v-model="coffeeTypeInfo.code" :disabled = dis></el-input>
        </el-form-item>
        <el-form-item label="图片" label-width="80px" required>
          <el-upload
            :on-success="handleAvatarSuccess"
            :show-file-list="false"
            :action="`${path}/fileUploadAndDownload/upload?noSave=1`"
            class="avatar-uploader"
            name="file"
          >
            <img :src="coffeeTypeInfo.image" class="avatar" v-if="coffeeTypeInfo.image" />
            <i class="el-icon-plus avatar-uploader-icon" v-else></i>
          </el-upload>
        </el-form-item>
      </el-form>
      <div class="dialog-footer" slot="footer">
        <el-button @click="closeAddCoffeeTypeDialog">取 消</el-button>
        <el-button @click="enterAddCoffeeTypeDialog('coffeeTypeInfo')" type="primary">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>


<script>
// 获取列表内容封装在mixins内部  getTableData方法 初始化已封装完成
const path = process.env.VUE_APP_BASE_API
import {
  getCoffeeTypeList,
  updateCoffeeType,
  addCoffeeType,
  delCoffeeType,
  getCoffeeTypeById
  
} from "@/api/coffee"
import infoList from "@/components/mixins/infoList"
export default {
  name: "coffeetype",
  mixins: [infoList],
  data() {
    const checkname = (rule, value, callback) => {
      if(!value) {
        return callback(new Error('咖啡类型名不能为空'))
      } else {
        callback()
      }
    }
    const checkcode = (rule, value, callback) => {
      if(!value) {
        return callback(new Error('咖啡类型编号不能为空'))
      } else {
        callback()
      }
    }       
    return {
      listApi: getCoffeeTypeList,
      listKey: "coffeetype",
      path: path,
      addCoffeeTypeDialog: false,
      isEdit: false,
      coffeeTypeInfo: {
        code: "",
        name: "",
        image: ""
      },
      dialogTitle: '',
      titleMap : {
        addData : "添加咖啡类型",
        updateData : "修改咖啡类型"
      },
      rules: {
        name: [{required: true, validator: checkname, trigger: 'blur' }],
        code:  [{required: true, validator: checkcode, trigger: 'blur' }]
      },
    }
  },
  methods: {
    async enterAddCoffeeTypeDialog(formName) {
      // eslint-disable-next-line no-console
      //console.log(this.coffeeInfo)
    this.$refs[formName].validate(async v => {
      let res
      if(this.isEdit) {
        res = await updateCoffeeType(this.coffeeTypeInfo)
      } else {
        res = await addCoffeeType(this.coffeeTypeInfo)
      }

      if (res.success) {
        this.$message({ type: "success", message: "创建成功" })
      }
      await this.getTableData()
      this.closeAddCoffeeDialog()
    })
  },
    closeAddCoffeeTypeDialog() {
      this.coffeeTypeInfo = {
        name: "",
        code: "",
        image: ""
      }
      this.addCoffeeTypeDialog = false
    },
    addCoffeeType() {
      this.coffeeTypeInfo = {
        name: "",
        code: "",
        image: ""
      }
      this.isEdit = false
      this.dis = false
      this.dialogTitle = 'addData'
      this.addCoffeeTypeDialog = true
    },
    handleAvatarSuccess(res) {
      this.coffeeTypeInfo.image = res.data.file.url;
    },
    async editCoffeeType(row) {
        const res = await getCoffeeTypeById({code: row.code})
        if(res.success){
            this.coffeeTypeInfo = res.data.coffeetype
        }
        this.isEdit = true
        this.dis = true
        this.dialogTitle = 'updateData'
        this.addCoffeeTypeDialog = true
    },
    async deleteCoffeeType(row) {
      const res = await delCoffeeType({ code: row.code })
      if(res.success) {
        this.$message({type: "success", message: "删除咖啡类型成功"})
      }
      await this.getTableData()
    }
  },
  created() {
      this.pageSize = 999
  },
}
</script>
<style scoped lang="scss">
.button-box {
  padding: 10px 20px;
  .el-button {
    float: right;
  }
}

.user-dialog {
  .avatar-uploader .el-upload:hover {
    border-color: #409eff;
  }
  .avatar-uploader-icon {
    border: 1px dashed #d9d9d9 !important;
    border-radius: 6px;
    font-size: 28px;
    color: #8c939d;
    width: 178px;
    height: 178px;
    line-height: 178px;
    text-align: center;
  }
  .avatar {
    width: 178px;
    height: 178px;
    display: block;
  }
}
</style>