<template>
  <v-container class="fill-height">
    <v-row justify="center">
      <v-col cols="10" sm="8" md="6" lg="5" class="text-center">
        <v-img src="../assets/img/logo.svg" max-width="35" class="ma-auto"></v-img>
        <p class="text--h6 font-weight-bold mt-4">登入Alphitter</p>

        <AlertErr :alertMsg.sync='alertMsg' v-if='alertMsg' />
        <v-form class="mt-4 mb-5" ref="form" v-model="valid" lazy-validation>
          <v-text-field single-line solo v-model.trim="email" :rules="[rules.required, rules.emailRules]" maxlength="50" label="帳號"></v-text-field>
          <v-text-field single-line solo v-model.trim="password" maxlength="20" :append-icon="showPassword ? 'mdi-eye' : 'mdi-eye-off'" :rules="[rules.required]" :type="showPassword ? 'text' : 'password'" label="密碼" @click:append="showPassword = !showPassword" 
          @keypress.13.prevent="signIn"></v-text-field>
          <v-btn rounded :disabled="!valid" color="primary" :loading="btnLoading" @click="signIn">
            登入
          </v-btn>
        </v-form>

        <div class="text-right">
          <router-link class="text-subtitle-2 secondary--text font-weight-bold" to="/signup">註冊</router-link><span class="grey--text"> | </span>
          <router-link class="text-subtitle-2 secondary--text font-weight-bold" to="/admin/signin">後台登入</router-link>
        </div>

      </v-col>
    </v-row>
  </v-container>
</template>
<script>
import authorizationAPI from "../apis/authorization";
import AlertErr from "@/components/AlertErr";

export default {
  name: "SignIn",
  components: { AlertErr },
  data: () => ({
    valid: true,
    email: "",
    password: "",
    showPassword: false,

    alertMsg: "",

    btnLoading: false,

    // 表單驗證條件
    rules: {
      required: (value) => !!value || "必填",
      emailRules: (value) =>
        /.+@.+\..+/.test(value) || "請使用Email登入，格式為：xxx@xxx.xx",
      // accountRules: (value) =>
      //   (value && value.length <= 50) || "帳號不得超過50個字",
    },
  }),
  methods: {
    async signIn() {
      // 表單驗證
      this.$refs.form.validate();
      // 讓按鈕不能重複按
      this.btnLoading = true;

      try {
        const response = await authorizationAPI.signIn({
          email: this.email,
          password: this.password,
        });
        this.btnLoading = false;

        // 取得 API 請求後的資料
        const { data } = response;
        // 登入失敗
        if (data.status === "error") {
          this.alertMsg = data.message;
          throw new Error(data.message);
        }
        // 登入成功
        this.alertMsg = "";
        localStorage.setItem("token", data.token);
        this.$router.push("/tweets");
      } catch (err) {
        this.btnLoading = false;
        console.log(err);
      }
    },
  },
  computed: {},
};
</script>