<template>
  <div class="app-container">
    <!-- 查询和其他操作 -->
    <div class="filter-container" style="margin: 10px 0 10px 0;">
      <el-input
        clearable
        class="filter-item"
        style="width: 200px;"
        v-model="keyword"
        placeholder="请输入管理员名"
      ></el-input>
      <el-button class="filter-item" type="primary" icon="el-icon-search" @click="handleFind">查找</el-button>
      <el-button class="filter-item" type="primary" @click="handleAdd" icon="el-icon-edit">添加</el-button>
    </div>

    <el-table :data="tableData" style="width: 100%">
      <el-table-column type="selection"></el-table-column>

      <el-table-column label="序号" width="60">
        <template slot-scope="scope">
          <span>{{scope.$index + 1}}</span>
        </template>
      </el-table-column>

      <el-table-column label="头像" width="120">
        <template slot-scope="scope">
          <img
            v-if="scope.row.photoList"
            :src="BASE_IMAGE_URL + scope.row.photoList[0]"
            style="width: 100px;height: 100px;"
          >
        </template>
      </el-table-column>

      <el-table-column label="用户名" width="100">
        <template slot-scope="scope">
          <span>{{ scope.row.userName }}</span>
        </template>
      </el-table-column>

      <el-table-column label="拥有角色" width="150">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.role" type="danger">{{scope.row.role.roleName}}</el-tag>
        </template>
      </el-table-column>

      <el-table-column label="性别" width="100">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.gender==1" type="success">男</el-tag>
          <el-tag v-if="scope.row.gender==2" type="danger">女</el-tag>
        </template>
      </el-table-column>

      <el-table-column label="登录次数" width="100">
        <template slot-scope="scope">
          <span>{{ scope.row.loginCount }}</span>
        </template>
      </el-table-column>

      <el-table-column label="登录IP" width="160">
        <template slot-scope="scope">
          <span>{{ scope.row.lastLoginIp }}</span>
        </template>
      </el-table-column>

      <el-table-column label="最后登录时间" width="160">
        <template slot-scope="scope">
          <span>{{ scope.row.lastLoginTime }}</span>
        </template>
      </el-table-column>

      <el-table-column label="状态" width="100">
        <template slot-scope="scope">
          <template v-if="scope.row.status == 1">
            <span>正常</span>
          </template>
          <template v-if="scope.row.status == 2">
            <span>推荐</span>
          </template>
          <template v-if="scope.row.status == 0">
            <span>已删除</span>
          </template>
        </template>
      </el-table-column>

      <el-table-column label="操作" fixed="right" min-width="150">
        <template slot-scope="scope">
          <el-button @click="handRest(scope.row)" type="warning" size="small">重置密码</el-button>
          <el-button @click="handleEdit(scope.row)" type="primary" size="small">编辑</el-button>
          <el-button @click="handleDelete(scope.row)" type="danger" size="small">删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <!--分页-->
    <div class="block">
      <el-pagination
        @current-change="handleCurrentChange"
        :current-page.sync="currentPage"
        :page-size="pageSize"
        layout="total, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>
    </div>

    <!-- 添加或修改对话框 -->
    <el-dialog :title="title" :visible.sync="dialogFormVisible">
      <el-form :model="form">
        <el-form-item label="头像" :label-width="formLabelWidth">
          <div class="imgBody" v-if="form.photoList">
            <i
              class="el-icon-error inputClass"
              v-show="icon"
              @click="deletePhoto()"
              @mouseover="icon = true"
            ></i>
            <img @mouseover="icon = true" @mouseout="icon = false" v-bind:src="BASE_IMAGE_URL + form.photoList[0]">
          </div>
          <div v-else class="uploadImgBody" @click="checkPhoto">
            <i class="el-icon-plus avatar-uploader-icon"></i>
          </div>
        </el-form-item>

        <el-form-item label="用户名" :label-width="formLabelWidth" required>
          <el-input v-model="form.userName"></el-input>
        </el-form-item>

        <el-form-item label="角色名" :label-width="formLabelWidth">
          <el-select v-model="form.roleUid" placeholder="请选择" style="width:150px">
            <el-option
              v-for="item in roleOptions"
              :key="item.uid"
              :label="item.roleName"
              :value="item.uid"
            ></el-option>
          </el-select>
        </el-form-item>

        <el-form-item label="昵称" :label-width="formLabelWidth">
          <el-input v-model="form.nickName"></el-input>
        </el-form-item>

        <el-form-item label="性别" :label-width="formLabelWidth" required>
          <template>
            <el-radio v-model="form.gender" label="1">男</el-radio>
            <el-radio v-model="form.gender" label="2">女</el-radio>
          </template>
        </el-form-item>

        <el-form-item label="邮箱" :label-width="formLabelWidth">
          <el-input v-model="form.email" ></el-input>
        </el-form-item>

        <el-form-item label="手机号" :label-width="formLabelWidth">
          <el-input v-model="form.mobile" ></el-input>
        </el-form-item>

        <el-form-item label="微信号" :label-width="formLabelWidth">
          <el-input v-model="form.weChat" ></el-input>
        </el-form-item>

        <el-form-item label="QQ号码" :label-width="formLabelWidth">
          <el-input v-model="form.qqNumber" ></el-input>
        </el-form-item>

        <el-form-item label="职业" :label-width="formLabelWidth">
          <el-input v-model="form.occupation" ></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitForm">确 定</el-button>
      </div>
    </el-dialog>

    <CheckPhoto
      @choose_data="getChooseData"
      @cancelModel="cancelModel"
      :photoVisible="photoVisible"
      :photos="photoList"
      :files="fileIds"
      :limit="1"
    ></CheckPhoto>
  </div>
</template>

<script>
import {
  getAdminList,
  addAdmin,
  editAdmin,
  deleteAdmin,
  restPwdAdmin,
  registerAdmin
} from "@/api/admin";

import { getRoleList } from "@/api/role";



import CheckPhoto from "../../components/CheckPhoto";

import { formatData } from "@/utils/webUtils";
export default {
  data() {
    return {
      BASE_IMAGE_URL: process.env.BASE_IMAGE_URL,
      tableData: [],
      roleOptions: [], //角色候选框
      loading: false, //搜索框加载状态
      roleData: [], //角色列表
      roleValue: [], //选择的角色列表
      keyword: "",
      currentPage: 1,
      pageSize: 10,
      total: 0, //总数量
      title: "增加标签",
      dialogFormVisible: false, //控制弹出框
      formLabelWidth: "120px",
      isEditForm: false,
      form: {},
      photoVisible: false, //控制图片选择器的显示
      photoList: [],
      fileIds: "",
      icon: false //控制删除图标的显示
    };
  },
  components: {
    CheckPhoto
  },
  created() {

    this.adminList();
    this.roleList();
  },
  methods: {
    adminList: function() {
      var params = new URLSearchParams();
      params.append("keyword", this.keyword);
      params.append("currentPage", this.currentPage);
      params.append("pageSize", this.pageSize);
      getAdminList(params).then(response => {
        if(response.code == "success") {
          console.log("得到的admin", response);
          this.tableData = response.data.records;
          this.currentPage = response.data.current;
          this.pageSize = response.data.size;
          this.total = response.data.total;
        }
      });
    },
    //角色远程搜索函数
    roleList: function () {
      var params = {};
      params.currentPage = 1;
      params.pageSize = 10;
      getRoleList(params).then(response => {
        this.roleOptions = response.data.records;
      });

    },
    //弹出选择图片框
    checkPhoto: function() {
      this.photoVisible = true;
      console.log(this.photoVisible);
    },
    getChooseData(data) {
      var that = this;
      this.photoVisible = false;
      this.photoList = data.photoList;
      this.fileIds = data.fileIds;
      var fileId = this.fileIds.replace(",", "");
      if (this.photoList.length >= 1) {
        this.form.fileUid = fileId;
        this.form.photoList = this.photoList;
      }
    },
    //关闭模态框
    cancelModel() {
      this.photoVisible = false;
    },
    deletePhoto: function() {
      console.log("点击了删除图片");
      this.form.photoList = null;
      this.form.fileUid = "";
      this.icon = false;
    },
    checkPhoto() {
      this.photoList = [];
      this.fileIds = "";
      this.photoVisible = true;
    },

    getFormObject: function() {
      var formObject = {
        uid: null,
        gender: "1"
      };
      return formObject;
    },
    handleFind: function() {
      this.adminList();
    },
    handleAdd: function() {
      this.dialogFormVisible = true;
      this.form = this.getFormObject();
      this.isEditForm = false;
    },
    handleEdit: function(row) {
      this.title = "编辑标签";
      this.dialogFormVisible = true;
      this.isEditForm = true;
      console.log(row);
      this.form = row;

      this.fileIds = this.form.avatar;

      this.roleValue = [];
      var roleList = [];
      //设置选择的角色列表
      if(row.roleList) {
        row.roleList.forEach(element => {
          roleList.push(element.uid);
        });
        this.roleValue = roleList;
      }

    },
    handRest: function(row) {
      var that = this;
      this.$confirm("此操作将会将该用户密码重置, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          let params = new URLSearchParams();
          params.append("uid", row.uid);
          restPwdAdmin(params).then(response => {
            if (response.code == "success") {
              that.$message({
                type: "success",
                message: response.data
              });
            } else {
              that.$message({
                type: "error",
                message: response.data
              });
            }
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除"
          });
        });
    },
    handleDelete: function(row) {
      var that = this;
      this.$confirm("此操作将该管理员删除, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          let params = new URLSearchParams();
          var adminUids = [];
          adminUids.push(row.uid);
          params.append("adminUids", adminUids);
          deleteAdmin(params).then(response => {
            console.log(response);
            this.$message({
              type: "success",
              message: response.data
            });
            that.adminList();
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除"
          });
        });
    },
    handleCurrentChange: function(val) {
      console.log("点击了换页");
      this.currentPage = val;
      this.adminList();
    },
    submitForm: function() {
      console.log("点击了提交表单", this.form);
      this.form.avatar = this.fileIds;
      if (this.isEditForm) {
        editAdmin(this.form).then(response => {
          console.log(response);
          if (response.code == "success") {
            this.$message({
              type: "success",
              message: response.data
            });
            this.dialogFormVisible = false;
            this.adminList();
          } else {
            this.$message({
              type: "error",
              message: response.data
            });
          }
        });
      } else {
        addAdmin(this.form).then(response => {
          console.log(response);
          if (response.code == "success") {
            this.$message({
              type: "success",
              message: response.data
            });
            this.dialogFormVisible = false;
            this.adminList();
          } else {
            this.$message({
              type: "error",
              message: response.data
            });
          }
        });
      }
    }
  }
};
</script>

<style>
.avatar-uploader .el-upload {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  margin: 0, 0, 0, 10px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}
.avatar-uploader .el-upload:hover {
  border-color: #409eff;
}
.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 100px;
  height: 100px;
  line-height: 100px;
  text-align: center;
}
.imgBody {
  width: 100px;
  height: 100px;
  border: solid 2px #ffffff;
  float: left;
  position: relative;
}
.uploadImgBody {
  margin-left: 5px;
  width: 100px;
  height: 100px;
  border: dashed 1px #c0c0c0;
  float: left;
  position: relative;
}
.uploadImgBody :hover {
  border: dashed 1px #00ccff;
}
.inputClass {
  position: absolute;
}
.img {
  width: 100%;
  height: 100%;
}
img {
  width: 100px;
  height: 100px;
}
</style>
