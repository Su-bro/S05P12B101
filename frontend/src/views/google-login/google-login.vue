<template>
  <div>
    <el-row>
      <el-col :span="8" :offset="8">
        <el-card :body-style="{ padding: '0px' }">
          <img :src="pic" class="googlelogin" />
          <div style="padding: 40px;">
            <h1>Google로 시작하기</h1>
            <div id="google-signin-btn"></div>
          </div>
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>
<script>
// import { ref } from "vue";
import pic from "@/assets/images/sample-img.png";
import axios from "../../common/lib/axios";
import $axios from "axios";

export default {
  data() {
    return {
      pic: pic
    };
  },
  mounted() {
    window.gapi.signin2.render("google-signin-btn", {
      onsuccess: this.onSignIn,
      width: 240,
      height: 50,
      longtitle: true,
      theme: "dark"
    });
  },
  methods: {
    onSignIn(googleUser) {
      const profile = googleUser.getBasicProfile();
      var self = this;
      console.log("ID: " + profile.getId());
      console.log("Full Name: " + profile.getName());
      console.log("Given Name: " + profile.getGivenName());
      console.log("Family Name: " + profile.getFamilyName());
      console.log("Image URL: " + profile.getImageUrl());
      console.log("Email: " + profile.getEmail());

      const id_token = googleUser.getAuthResponse().id_token;
      const access_token = googleUser.getAuthResponse().access_token;
      let token = id_token;

      // http post 요청으로 서버에 id토큰 유효성 검사
      const CLIENT_ID =
        "890408784203-ko60b9fublcta8prgu5lll4ccpilqsoo.apps.googleusercontent.com";
      const { OAuth2Client } = require("google-auth-library");
      const client = new OAuth2Client(CLIENT_ID);
      async function verify() {
        const ticket = await client.verifyIdToken({
          idToken: token,
          audience: CLIENT_ID
        });
        const payload = ticket.getPayload();
        const userid = payload["sub"];
        // 회원가입 + 로그인
        const userInfo = {
          nickname: profile.getName(),
          email: profile.getEmail()
        };
        //로그인 기능
        var email = "";
        var nickname = "";

        $axios.post("/users/glogin", userInfo).then(res => {
          if (res.status === 200) {
            console.log("구글 계정 정보, 요청 전달 성공 ");
            console.log("백엔드로부터 전달 받은 response :" + res);
            console.log(
              "ResponseEntity/data/guserinfo : " + res.data.guserinfo
            );
            email = res.data.guserinfo.email;
            nickname = res.data.guserinfo.nickname;

            sessionStorage.setItem(
              "userInfo",
              JSON.stringify({
                nickname: nickname,
                email: email,
                imgUrl: profile.getImageUrl()
              })
            );
            console.log(
              nickname + "님 환영합니다 \n 당신의 이메일주소는" + email
            );
            // 화살표 함수 안에서는 this참조가 되지않기때문에 self 등록해준다.
            self.$store.commit("root/setLogin", true);
          } else {
            alert("서버와 연결이 불안정합니다");
          }
        });
      }
      verify().catch(console.error);
    }
  }
};
</script>
<style scoped>
.googlelogin {
  /* height: 300px; */
  width: 100%;
}
</style>
