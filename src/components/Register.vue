<template>
  <!--background-color: #f0f2f5; background-image: linear-gradient(0deg,#04c0c6,#2ba3de 51%,#835be3); -->
  <div>
    <el-form :model="user" :rules="rules" ref="user">


      <el-form-item prop="userName">
        <el-input prefix-icon="el-icon-user" v-model.trim="user.userName" placeholder="User Name"></el-input>
      </el-form-item>
      <el-form-item prop="password">
        <el-input
          prefix-icon="el-icon-key"
          type="password"
          v-model.trim="user.password"
          placeholder="Password"
          show-password
        ></el-input>
      </el-form-item>
      <el-form-item prop="email">
        <el-input
          v-model.trim="user.email"
          
          placeholder="Email Address"
        ></el-input>
      </el-form-item>
            <el-form-item prop="mobile">
        <el-input
          v-model.trim="user.mobile"
          
          placeholder="Mobile Number"
        ></el-input>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" style="width: 100%;" @click="Register('user')">Register</el-button>
      </el-form-item>
      <el-form-item v-if="authorization.enabled">
        <el-button type="success" style="width: 100%;">
          <el-link
            :href="authorization.server + authorization.siteKey"
            :underline="false"
            style="color: #fff"
          >{{ authorization.companyName }} Sign In</el-link>
        </el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
import API from "@/api/api.js";
import { store } from "@/vuex/store.js";
import message from "@/utils/message.js";
import { isEmpty, validateEmail, validateMobile } from "@/utils/validate.js";
export default {
  data() {
           var validateEmailFormat = (rule, value, callback) => {
      if (!isEmpty(value) && !validateEmail(value)) {
        return callback(new Error("Incorrect format"));
      }
      callback();
    };
    var validateMobileFormat = (rule, value, callback) => {
      if (!isEmpty(value) && !validateMobile(value)) {
        return callback(new Error("Incorrect format"));
      }
      callback();
    };
    var validateUserName = (rule, value, callback) => {
      let url = "/user/validateUserName/" + value;
      API.get(
        url,
        null,
        response => {
          let result = response.data;
          if (result.code == 0) {
            callback();
          } else if (this.user.userId !== result.data.userId) {
            callback(new Error(value + " has been used"));
          } else {
            callback();
          }
        },
        err => {
          return callback(new Error("Network error"));
        }
      );
    };
    return {

      user: {},
      authorization: {},
      rules: {
        userRole: [
          {
            required: true,
            message: "Please select user role",
            trigger: "blur"
          }
        ],
        userName: [
          { required: true, message: "Please enter user name", trigger: "blur" },
                    {
            required: true,
            validator: validateUserName,
            trigger: "blur"
          }
        ],
        password: [
          { required: true, message: "Please enter password", trigger: "blur" }
        ],
        
                email: [
          {
            
            trigger: "blur"
          }
        ],
        mobile: [
          {
            
            trigger: "blur"
          }
        ]
      },
      activeName: "second"
    };
  },
  methods: {

    
    Register(user) {
  
console.log(this.user);
        this.$refs[user].validate(valid => {
        if (valid) {
          this.user.groupId = 1;
          this.user.userRole=0;
          console.log(this.user);
          let url = "/user/addUser";
          
          API.post(
            url,
            this.user,
            response => {
              let result = response.data;
              if (result.code == 0) {
                this.$refs[user].resetFields();
                message.success("successfully registered");
              } else {
                message.error("register failed");
              }
              
            },
            err => {
              
              message.error(err);
            }
          );
        }
      });

    }

    // getUserFromSession() {
    //   let url = "/user/getUserFromSession";
    //   API.get(
    //     url,
    //     null,
    //     response => {
    //       let result = response.data;
    //       if (result.code == 0) {
    //         store.dispatch("setUser", result.data);
    //         this.$router.push({ name: "index" });
    //       }
    //     },
    //     err => {
    //       message.error("Auto get user failed.");
    //     }
    //   );
    // }
  },

  mounted() {
    //this.getUserFromSession();
    // store.dispatch("setUser", {
    //   userRole: 2
    // });
    // store.dispatch("setCurrentGroup", {});
    // store.dispatch("setGroupList", []);
  }
};
</script>

<style scoped>
/* .link-wrapper {
  display: flex;
  justify-content: center;
  font-size: 12px;
  color: #909399;
}
.doc-link {
  font-size: 12px;
} */
</style>
