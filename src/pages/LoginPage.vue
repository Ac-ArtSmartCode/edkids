<template>
  <div class="row justify-center items-center">
    <div class="col-sm-8 text-center">
      <q-img width="150px" src="../assets/logo_edkids.png"></q-img>
    </div>
  </div>
  <div class="row q-mt-sm justify-start q-pl-lg tab">
    <div class="col-sm-5 col-5">
      <q-tabs v-model="tab" indicator-color="white" class="text-teal">
        <q-tab class="q-px-sm" name="mails" label="นักเรียน" />
        <q-tab class="q-px-sm" name="alarms" label="คุณครู" />
      </q-tabs>
    </div>
  </div>

  <div class="row justify-center items-start">
    <q-form @submit="onSubmit">
      <div class="col-sm-12">
        <q-tab-panels class="transparent no-padding fit" v-model="tab" animated>
          <q-tab-panel name="mails">
            <div class="row fit justify-center">
              <div class="col-sm-11 col-7">
                <q-input
                  dark
                  v-model="stdCode"
                  :rules="[codeInvalid]"
                  dense
                  color="white"
                  label="รหัสนักศึกษา"
                ></q-input>
              </div>
              <div class="col-sm-11 col-7">
                <q-input
                  label="วัน/เดือน/ปี เกิด"
                  mask="##/##/####"
                  fill-mask
                  color="white"
                  v-model="birth"
                  dark
                  dense
                  :rules="[birthInvalid]"
                  class="text-h6"
                  hint="หากวันเกิดหรือเดือนเกิดเป็นเลขเดียวให้เติมเลข '0'"
                />
              </div>
            </div>
          </q-tab-panel>
          <q-tab-panel name="alarms">
            <div class="row fit justify-center">
              <div class="col-sm-11 col-7">
                <q-input
                  v-model="email"
                  dark
                  dense
                  label="ชื่อผู้ใช้"
                  color="white"
                  :rules="[emailInvalid]"
                ></q-input>
              </div>
              <div class="col-sm-11 col-7">
                <q-input
                  v-model="password"
                  :rules="[passwordInvalid]"
                  dark
                  dense
                  :type="show ? 'password' : 'text'"
                  label="รหัสผ่าน"
                  color="white"
                >
                  <template v-slot:append>
                    <q-btn
                      :icon="show ? 'visibility' : 'visibility_off'"
                      dense=""
                      @click="show = !show"
                      flat=""
                    ></q-btn>
                  </template>
                </q-input>
              </div>
            </div>
          </q-tab-panel>
        </q-tab-panels>
        <div class="row justify-center">
          <div class="col-sm-10 q-mt-md">
            <q-btn
              style="background-color: #60f4a0"
              class="q-mx-auto fit text-blue-grey-14 text-weight-bold"
              label="เข้าสู่ระบบ"
              type="submit"
            ></q-btn>
          </div>
        </div>
      </div>
    </q-form>
  </div>
  <div class="row justify-center q-mt-md">
    <div class="col-sm-10 col-11 z-max text-center text-white">
      <p>
        หากท่าพบปัญหาการใช้งานหรือต้องการสมัครการเข้าใช้งานสำหรับครูผู้ควบคุม
        โปรด
        <a href="https://line.me/ti/p/9MYGyI-W25" class="text-primary">
          คลิ๊ก</a
        >
        เพื่อติดต่อ
      </p>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";
import { Loading, QSpinnerGears, useMeta } from "quasar";
import { AuthenApi } from "src/api/AuthenApi";
import { alertShow } from "src/composable/alertShow";
import { useAuthenStore } from "src/stores/authen";
useMeta({ title: "Login" });
const { alertDanger, alertSuccess } = alertShow();
const { loginProcess } = AuthenApi();

const authenStore = useAuthenStore();
const tab = ref("mails");
const stdCode = ref("");
const show = ref(true);
const birth = ref("");
const email = ref("");
const teacherData = ref();
const password = ref("");
const studentData = ref();
const emailInvalid = (val) => !!val || "กรุณากรอกอีเมลล์";
const passwordInvalid = (val) => !!val || "กรุณาใส่รหัสผ่าน";
const codeInvalid = (val) => !!val || "กรุณากรอกรหัสนักเรียน";
const birthInvalid = (val) => !!val || "กรุณากรอกวันเดือนปีเกิด";

const onSubmit = () => {
  if (tab.value == "alarms") {
    teacherLogin();
  } else if (tab.value == "mails") {
    userSubmit();
  }
};

const teacherLogin = async () => {
  Loading.show({
    spinner: QSpinnerGears,
  });
  const response = await loginProcess({
    _u: email.value,
    _p: password.value,
  });

  Loading.hide();

  if (response && response.userData?.apiKey && response.userData.teacher_id) {
    teacherData.value = response.userData;
    authenStore.setAuthen(teacherData.value);

    alertSuccess("เข้าสู่ระบบสำเร็จ", "ยินดีต้อนรับคุณครู");
    setTimeout(() => {
      window.location.replace("/");
    }, 800);

    // console.log(teacherData.value);
  } else if (response.status != true) {
    alertDanger(
      "เข้าสู่ระบบไม่สำเร็จ",
      "กรุณาตรวจสอบชื่อผู้ใช้และรหัสผ่านให้ถูกต้อง"
    );
  } else {
    alertDanger(
      "เข้าสู่ระบบไม่สำเร็จ",
      "กรุณาตรวจสอบชื่อผู้ใช้และรหัสผ่านให้ถูกต้อง"
    );
  }
};
// user login part

const userSubmit = async () => {
  // cut / in password

  const day = birth.value.substring(0, 2);
  const month = birth.value.substring(3, 5);
  const year = birth.value.substring(6, 10);
  const result = year + month + day;

  Loading.show({
    spinner: QSpinnerGears,
  });
  const response = await loginProcess({
    _u: stdCode.value,
    _p: result,
  });
  Loading.hide();
  // console.log(response);
  // if logined success
  if (response && response.userData?.std_id && response.userData.apiKey) {
    studentData.value = response.userData;
    authenStore.setUserAuthen(studentData.value);
    // $q.notify({
    //   message: "Login Success!!",
    //   // avatar: response.userData.picture.path,
    // });
    alertSuccess(
      "เข้าสู่ระบบสำเร็จ",
      "ยินดีต้อนรับผู้ปกครองเรายินดีดูแลบุตรหลานของท่าน"
    );
    setTimeout(() => {
      window.location.replace("#/user");
    }, 500);
  } else if (response.status != true) {
    alertDanger(
      "เข้าสู่ระบบไม่สำเร็จ",
      "กรุณาตรวจสอบชื่อผู้ใช้และรหัสผ่านให้ถูกต้อง"
    );
  } else {
    alertDanger(
      "เข้าสู่ระบบไม่สำเร็จ",
      "กรุณาตรวจสอบชื่อผู้ใช้และรหัสผ่านให้ถูกต้อง"
    );
  }
};
</script>

<style lang="scss" scoped>
body.screen--xs {
  .tab {
    margin-left: 4rem;
  }
}
</style>
