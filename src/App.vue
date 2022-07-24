<template>

  <div class="step_box" :class="step_1" name="step_1">
    <div class="input_wrap">
      <div class="user_info_box">
        <label for="user_email">이메일</label>
        <input id="user_email" type="text" ref="user_id" :value="user_id"/>
      </div>
    </div>
    <div class="input_wrap">
      <div class="user_info_box">
        <label for="user_password">비밀번호</label>
        <input id="user_password" type="password" ref="user_password" :value="user_password"/>
      </div>
    </div>
    <div class="input_wrap">
      <div class="user_info_box">
        <label for="user_password_confirm">비밀번호 확인</label>
        <input id="user_password_confirm" type="password" ref="user_password_confirm" :value="user_password_confirm"/>
      </div>
    </div>

    <div class="footer_btn_wrap">
      <button class="step_btn" @click="step_1_check">다음</button>
    </div>
  </div>
  <!-- step_1 -->

  <div class="step_box" :class="step_2" name="step_2">
    <div class="input_wrap">
      <div class="user_info_box">
        <label for="user_name">이름</label>
        <input id="user_name" type="text" ref="user_name" :value="user_name"/>
      </div>
    </div>
    <div class="input_wrap">
      <div class="user_info_box">
        <label for="user_phone">연락처</label>
        <input id="user_phone" type="number" ref="user_phone" :value="user_phone"/>
      </div>
    </div>
    <div class="input_wrap">
      <div class="user_info_box">
        <label>주소</label>
        <button class="search_addr_btn" @click="call_post_code">우편번호</button>
      </div>
      <div class="addr_box">
        <input id="user_postcode" type="text" ref="user_postcode" :value="user_postcode" readonly/>
      </div>
      <div class="addr_box">
        <input id="user_addr" type="text" ref="user_addr" :value="user_addr" readonly/>
      </div>
    </div>

    <div class="footer_btn_wrap">
      <button class="step_btn" @click="step_remote('step_1')">이전</button>
      <button class="step_btn" @click="step_2_check">다음</button>
    </div>
  </div>
  <!-- step_2 -->

  <div class="step_box" :class="step_3" name="step_3">
    <div class="input_wrap card">
      <label for="card_number">카드번호</label>
      <div class="user_card_box">
        <input id="card_number" class="card_number_box" ref="card_1" :value="card_1"/>
        <input class="card_number_box" ref="card_2" :value="card_2"/>
        <input class="card_number_box" ref="card_3" :value="card_3"/>
        <input class="card_number_box" ref="card_4" :value="card_4"/>
      </div>
    </div>

    <div class="footer_btn_wrap">
      <button class="step_btn" @click="join_submit">완료</button>
    </div>
  </div>
  <!-- step_3 -->


  <div class="step_box" :class="step_4">
    <p class="complete_text">
      '{{user_name}}'님<br/>
      회원가입 되었습니다.
    </p>
    <p>이메일: {{user_email}}</p>
    <p>주소: {{user_addr}}</p>
    <p>연락처: {{user_phone}}</p>
  </div>
  <!-- step_4 -->



  <div id="daum_post_wrap" :class="daum_post_remote" ref="embed"></div>

</template>

<script>




export default {
  name: 'App',
  data(){
    return {
      step_1: "on",
      step_2: "",
      step_3: "",
      step_4: "",
      user_id: "",
      user_password: "",
      user_password_confirm: "",
      user_name: "",
      user_phone: "",
      daum_post_remote: "",
      user_postcode: "",
      user_addr: "",
      card_1: "",
      card_2: "",
      card_3: "",
      card_4: ""
    }
  },

  methods : {
    step_remote(step) {
      this.step_1 = "";
      this.step_2 = "";
      this.step_3 = "";

      if(step == 'step_1') {
        this.step_1 = "on";
      } else if(step == 'step_2') {
        this.step_2 = "on";
      } else {
        this.step_3 = "on";
      }
    },
    step_1_check() {
      this.user_id = this.$refs.user_id.value;
      this.user_password = this.$refs.user_password.value;
      this.user_password_confirm = this.$refs.user_password_confirm.value;

      if(this.user_id != '' && this.user_password != '' && this.user_password_confirm != '') {
        if(this.email_reg_check(this.user_id)) {
          if(this.password_reg_check(this.user_password)) {
            if(this.user_password == this.user_password_confirm) {
              //next
              this.step_remote('step_2');
            }
          }
        }
      }
    },
    email_reg_check(value) {
      const reg = /^[0-9a-zA-Z]([-_.]?[0-9a-zA-Z])*@[0-9a-zA-Z]([-_.]?[0-9a-zA-Z])*.[a-zA-Z]{2,3}$/i;
      if(reg.test(value)) {
        return true;
      } else {
        return false;
      }
    },
    password_reg_check(value) {
      const reg = /^(?=.*[A-Za-z])(?=.*\d)(?=.*[@$!%*#?&])[A-Za-z\d@$!%*#?&]{8,}$/i;
      if(reg.test(value)) {
        return true;
      } else {
        return false;
      }
    },


    call_post_code() {
      this.daum_post_remote = "on";
      new window.daum.Postcode({
        oncomplete: (data) => {
            // 팝업에서 검색결과 항목을 클릭했을때 실행할 코드를 작성하는 부분.

            // 도로명 주소의 노출 규칙에 따라 주소를 조합한다.
            // 내려오는 변수가 값이 없는 경우엔 공백('')값을 가지므로, 이를 참고하여 분기 한다.
            let fullRoadAddr = data.roadAddress; // 도로명 주소 변수
            let extraRoadAddr = ''; // 도로명 조합형 주소 변수

            // 법정동명이 있을 경우 추가한다. (법정리는 제외)
            // 법정동의 경우 마지막 문자가 "동/로/가"로 끝난다.
            if(data.bname !== '' && /[동|로|가]$/g.test(data.bname)){
                extraRoadAddr += data.bname;
            }
            // 건물명이 있고, 공동주택일 경우 추가한다.
            if(data.buildingName !== '' && data.apartment === 'Y'){
              extraRoadAddr += (extraRoadAddr !== '' ? ', ' + data.buildingName : data.buildingName);
            }
            // 도로명, 지번 조합형 주소가 있을 경우, 괄호까지 추가한 최종 문자열을 만든다.
            if(extraRoadAddr !== ''){
                extraRoadAddr = ' (' + extraRoadAddr + ')';
            }
            // 도로명, 지번 주소의 유무에 따라 해당 조합형 주소를 추가한다.
            if(fullRoadAddr !== ''){
                fullRoadAddr += extraRoadAddr;
            }

            // 우편번호와 주소 정보를 해당 필드에 넣는다.
            this.user_postcode = data.zonecode; //5자리 새우편번호 사용
            this.user_addr = fullRoadAddr;
            this.daum_post_remote = "";
        }
      }).embed(this.$refs.embed)
    },

    step_2_check() {
      this.user_name = this.$refs.user_name.value;
      this.user_phone = this.$refs.user_phone.value;
      this.user_postcode = this.$refs.user_postcode.value;
      this.user_addr = this.$refs.user_addr.value;

      if(this.user_name != '' && this.user_phone != '' && this.user_postcode != '' && this.user_addr != '') {
        //next
        this.step_remote('step_3');
      }
    },
    phone_reg_exp(value) {
      const reg = /^01([0|1|6|7|8|9])-?([0-9]{3,4})-?([0-9]{4})$/i;
      if(reg.test(value)) {
        return true;
      } else {
        return false;
      }
    },

    join_submit() {
      this.card_1 = this.$refs.card_1.value;
      this.card_2 = this.$refs.card_2.value;
      this.card_3 = this.$refs.card_3.value;
      this.card_4 = this.$refs.card_4.value;

      console.log(this.card_1.length);
    }
  },

  components: {
    
  }
}
</script>

<style>
.step_box {
  display: none; width: 400px; height: 300px; margin: 0 auto; border: 1px solid #000; box-sizing: border-box; padding: 10px; position: relative;
}

.step_box.on {
  display: block;
}

.step_box .input_wrap {
  margin-top: 10px;
}

.step_box .input_wrap:first-child {
  margin-top: 0;
}

.step_box .user_info_box {
  display: flex; justify-content: space-between;
}

.step_box .addr_box {
  display: flex; margin-top: 10px;
}

.step_box .addr_box input[type="text"] {
  display: block; width: 100%;
}


.step_box .footer_btn_wrap {
  position: absolute; left: 10px; bottom: 10px; width: calc(100% - 20px); box-sizing: border-box; display: flex; justify-content: flex-end;
}

.step_box[name="step_2"] .footer_btn_wrap {
  justify-content: space-between;
}

.user_card_box {
  display: flex; justify-content: space-between;
}

.user_card_box input {
  display: block; width: 20%;
}

.complete_text {
  text-align: center;
}

#daum_post_wrap {
  position: fixed; top: 50%; left: 50%; transform: translate(-50%, -50%); width: 500px; border: 2px solid #000; display: none;
}

#daum_post_wrap.on {
  display: block;
}
</style>
