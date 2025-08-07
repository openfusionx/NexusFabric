<template>
  <div>
    <common-layout>
      <div class="top">
        <div class="header">
          <!-- <img src="@/assets/img/logo.png" alt /> -->
          <!-- <span class="logo-name">DataSophon</span> -->
        </div>
      </div>
      <div class="login">
        <div class="icon-back" v-if="isLogin" @click="isLogin = false">
          <svg
            xmlns="http://www.w3.org/2000/svg"
            width="11"
            height="18"
            viewBox="0 0 11 18"
            fill="none"
          >
            <path
              d="M9.75 16.5L2.25 9L9.75 1.5"
              stroke="white"
              stroke-width="2"
            />
          </svg>
        </div>
        <div class="login-title">
          <img src="@/assets/img/login-logo.png" alt class="header-logo" />
        </div>
        <div class="login-logo">
          <img src="@/assets/img/login-text.png" class="header-text" />
        </div>
        <div v-if="!isLogin" class="login-container">
          <div class="login-title-text">选择您想要使用的服务</div>
          <div class="login-list">
            <div class="login-list-text">监控服务</div>
            <div class="login-list-circle" @click="changeLoginState('1')">
              <svg
                xmlns="http://www.w3.org/2000/svg"
                width="17"
                height="18"
                viewBox="0 0 17 18"
                fill="none"
              >
                <path
                  d="M8 16.5L15.5 9L8 1.5"
                  stroke="white"
                  stroke-width="2"
                />
                <path
                  d="M15.5 8.99893L0.5 8.99893"
                  stroke="white"
                  stroke-width="2"
                />
              </svg>
            </div>
          </div>
          <div class="login-list">
            <div class="login-list-text">管理服务</div>
            <div class="login-list-circle" @click="changeLoginState('2')">
              <svg
                xmlns="http://www.w3.org/2000/svg"
                width="17"
                height="18"
                viewBox="0 0 17 18"
                fill="none"
              >
                <path
                  d="M8 16.5L15.5 9L8 1.5"
                  stroke="white"
                  stroke-width="2"
                />
                <path
                  d="M15.5 8.99893L0.5 8.99893"
                  stroke="white"
                  stroke-width="2"
                />
              </svg>
            </div>
          </div>
        </div>
        <a-form @submit="onSubmit" :form="form" v-else>
          <a-alert
            type="error"
            :closable="true"
            v-show="error"
            :message="error"
            showIcon
            style="margin-bottom: 24px"
          />
          <a-form-item label="用户名">
            <a-input
              class="login-input"
              size="large"
              placeholder="请输入用户名"
              v-decorator="[
                'name',
                {
                  rules: [
                    {
                      required: true,
                      message: '请输入用户名',
                      whitespace: true,
                    },
                  ],
                },
              ]"
            ></a-input>
          </a-form-item>
          <a-form-item label="密码">
            <a-input
              class="login-input"
              size="large"
              placeholder="请输入密码"
              type="password"
              v-decorator="[
                'password',
                {
                  rules: [
                    { required: true, message: '请输入密码', whitespace: true },
                  ],
                },
              ]"
            ></a-input>
          </a-form-item>
          <a-form-item>
            <a-button
              :loading="logging"
              style="
                width: 320px;
                height: 60px;
                border-radius: 30px;
                margin-top: 47px;
                font-size: 18px;
              "
              size="large"
              htmlType="submit"
              type="primary"
              >登录</a-button
            >
          </a-form-item>
        </a-form>
      </div>
    </common-layout>
  </div>
</template>

<script>
import PageFooter from "@/layouts/footer/PageFooter";

import CommonLayout from "@/layouts/CommonLayout";
import { login, getRoutesConfig } from "@/services/user";
import { setAuthorization } from "@/utils/request";
import { loadRoutes } from "@/utils/routerUtil";
import { mapMutations } from "vuex";
import { mapState } from "vuex";

export default {
  name: "Login",
  components: { CommonLayout },
  data() {
    return {
      logging: false,
      error: "",
      form: this.$form.createForm(this),
      isLogin: false,
      loginType: "",
    };
  },
  computed: {
    systemName() {
      return this.$store.state.setting.systemName;
    },
    ...mapState("setting", ["footerLinks", "copyright"]),
  },
  methods: {
    ...mapMutations("account", ["setUser", "setPermissions", "setRoles"]),
    onSubmit(e) {
      e.preventDefault();
      this.form.validateFields((err) => {
        if (!err) {
          this.logging = true;
          const username = this.form.getFieldValue("name");
          const password = this.form.getFieldValue("password");
          this.$axiosPost(global.API.login, { username, password }).then(
            (res) => this.afterLogin(res)
          );
        }
      });
    },
    afterLogin(res) {
      this.logging = false;
      const loginRes = res.data;
      if (res.code === 200) {
        setAuthorization({ sessionId: loginRes.sessionId });
        this.setUser(res.userInfo);
        loadRoutes();
        this.$store.commit("setting/setIsCluster", "");
        if (this.loginType == "2") {
          this.$router.push("/colony-manage/colony-list");
        } else {
          this.$router.push("/overview");
        }
        this.$message.success("登录成功", 3);
      }
    },
    changeLoginState(state) {
      this.loginType = state;
      this.isLogin = true;
    },
  },
};
</script>

<style lang="less" scoped>
.common-layout {
  .top {
    text-align: center;
    .header {
      padding: 0 0 0 40px;
      height: 44px;
      line-height: 44px;
      display: flex;
      justify-items: center;
      align-items: center;
      a {
        text-decoration: none;
      }
      .logo {
        height: 44px;
        vertical-align: top;
        margin-right: 16px;
      }
      .logo-name {
        font-size: 24px;
        color: #333333;
        letter-spacing: 0;
        text-align: center;
        font-weight: 600;
        padding-left: 10px;
      }
      .title {
        font-size: 33px;
        color: @title-color;
        // font-family: "Myriad Pro", "Helvetica Neue", Arial, Helvetica,
        // sans-serif;
        font-weight: 600;
        position: relative;
        top: 2px;
      }
    }
    .desc {
      font-size: 14px;
      color: @text-color-second;
      margin-top: 12px;
      margin-bottom: 40px;
    }
  }
  .login {
    text-align: center;
    position: absolute;
    right: 60px;
    width: 640px;
    height: 700px;
    // height: 644px;
    background: #ffffff;
    box-shadow: 0px 2px 10px 0px rgba(0, 0, 0, 0.1);
    border-radius: 4px;
    top: 50%;
    transform: translateY(-50%);
    // display: flex;
    // flex-direction: column;
    // align-items: center;
  }
}
.header-logo {
  width: 90px;
  height: 50px;
  margin-top: 86px;
}
.header-text {
  width: 201px;
  height: 28px;
  margin-top: 43px;
  margin-bottom: 78px;
}
.login-list {
  width: 400px;
  height: 140px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0 60px;
  background: #f5f8f9;
  margin-bottom: 32px;
}
.login-list:hover {
  border: 1px solid #4274ad;
  background: #e2f1ff;
  cursor: pointer;
}
.login-title {
  padding: 0 160px;
}
.login-title-text {
  font-size: 16px;
  font-style: normal;
  font-weight: 400;
  color: #000;
  margin-bottom: 26px;
}
.login-list-circle {
  width: 36px;
  height: 36px;
  border-radius: 50%;
  background: #3e6eaa;
  display: flex;
  align-items: center;
  justify-content: center;
}
.login-list-text {
  font-size: 26px;
  color: #000;
  font-weight: 400;
}
.icon-back {
  width: 36px;
  height: 36px;
  background: #c3c5cf;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  position: absolute;
  top: 40px;
  left: 40px;
}
.login-container {
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
}
.icon-back:hover {
  cursor: pointer;
}
/deep/ .ant-input {
  height: 60px;
}
/deep/ .ant-form-item .ant-form-item-required:before {
  display: none;
}
/deep/ .ant-form {
  text-align: left;
  padding: 0 160px;
}
</style>
