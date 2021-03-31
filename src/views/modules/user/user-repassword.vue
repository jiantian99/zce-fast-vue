<template>
  <el-dialog
    :title="'修改密码'"
    :close-on-click-modal="false"
    :visible.sync="visible"
  >
    <el-form
      :model="dataForm"
      :rules="dataRule"
      ref="dataForm"
      @keyup.enter.native="dataFormSubmit()"
      label-width="80px"
    >
      <el-form-item label="用户id" prop="userId">
        <el-input
          v-model="dataForm.userId"
          placeholder="用户id"
          :disabled="true"
        ></el-input>
      </el-form-item>
      <el-form-item label="用户名" prop="username">
        <el-input
          v-model="dataForm.username"
          placeholder="用户名"
          :disabled="true"
        ></el-input>
      </el-form-item>
      <el-form-item label="新密码" prop="password">
        <el-input v-model="dataForm.password" placeholder="新密码"></el-input>
      </el-form-item>
      <el-form-item label="密码确认" prop="repassword">
        <el-input
          v-model="dataForm.repassword"
          placeholder="密码确认"
        ></el-input>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
import crypto from "crypto";
export default {
  data() {
    return {
      visible: false,
      dataForm: {
        id: 0,
        userId: "",
        username: "",
        password: "",
        repassword: "",
      },
      dataRule: {
        // userId: [
        //   { required: true, message: '用户id不能为空', trigger: 'blur' }
        // ],
        password: [
          { required: true, message: "密码不能为空", trigger: "blur" },
        ],
        repassword: [
          { required: true, message: "密码不能为空", trigger: "blur" },
        ],
      },
    };
  },
  methods: {
    init(id) {
      this.dataForm.id = id || 0;
      this.visible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].resetFields();
        if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl(`/user/user/info/${this.dataForm.id}`),
            method: "get",
            params: this.$http.adornParams(),
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.dataForm.userId = data.user.userId;
              this.dataForm.username = data.user.username;
            }
          });
        }
      });
    },
    // 表单提交
    dataFormSubmit() {
      var md5 = crypto.createHash("md5");
      md5.update(this.dataForm.password);
      var pwd = md5.digest("hex");
      // console.log(password);
      this.$refs["dataForm"].validate((valid) => {
        if (valid) {
          this.$http({
            url: this.$http.adornUrl(`/user/user/update`),
            method: "post",
            data: this.$http.adornData({
              id: this.dataForm.id || undefined,
              userId: this.dataForm.userId,
              // password: this.dataForm.password,
              password: pwd,
            }),
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.$message({
                message: "操作成功",
                type: "success",
                duration: 1500,
                onClose: () => {
                  this.visible = false;
                  this.$emit("refreshDataList");
                },
              });
            } else {
              this.$message.error(data.msg);
            }
          });
        }
      });
    },
  },
};
</script>
