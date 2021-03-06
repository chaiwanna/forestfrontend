<template>
  <b-form @submit.stop.prevent="onSubmit" class="login-box" data-cy="login-form">
    <b-form-group id="group-username" label label-for="input-username" description>
      <b-form-input
        id="input-username"
        v-model="form.username"
        type="text"
        required
        placeholder="ชื่อผู้ใช้งาน"
        :state="$v.form.username.$dirty ? !$v.form.username.$error : null"
        data-cy="login-username"
        class="from-input input-username"
        :class="{ 'border-danger': errorMessages }"
      ></b-form-input>

      <b-form-invalid-feedback
        id="input-username-invalid"
        data-cy="login-username-invalid"
        class="invalid-feedback invalid-feedback-username"
      >โปรดกรอกชื่อผู้ใช้งาน</b-form-invalid-feedback>
    </b-form-group>

    <b-form-group id="group-password" label label-for="input-password" description>
      <b-form-input
        id="input-password"
        type="password"
        v-model="form.password"
        required
        placeholder="รหัสผ่าน"
        :state="$v.form.password.$dirty ? !$v.form.password.$error : null"
        data-cy="login-password"
        class="from-input input-password"
        :class="{ 'border-danger': errorMessages }"
      ></b-form-input>

      <b-form-invalid-feedback
        id="input-password-invalid"
        data-cy="login-password-invalid"
        class="invalid-feedback invalid-feedback-password"
      >โปรดกรอกรหัสผ่าน</b-form-invalid-feedback>
    </b-form-group>

    <template v-if="successMessages || errorMessages">
      <b-row class="mb-2">
        <b-col
          v-if="successMessages"
          data-cy="login-success-message"
          class="text-primary message-col"
        >
          {{
          successMessages
          }}
        </b-col>
        <b-col v-if="errorMessages" data-cy="login-error-message" class="text-danger message-col">
          {{
          errorMessages
          }}
        </b-col>
      </b-row>
    </template>

    <b-row>
      <b-col>
        <b-button
          data-cy="login-button"
          class="btn btn-success btn-block"
          type="submit"
          variant="primary"
          :disabled="$v.form.$invalid || loading"
        >
          <span class="spinner spinner-white" v-if="loading"></span>
          เข้าสู่ระบบ
        </b-button>
        <b-button :to="{path:`/register/new${isQA}`}" class="btn btn-secondary btn-block">ลงทะเบียน</b-button>
      </b-col>
    </b-row>
  </b-form>
</template>

<script>
import { mapState, mapGetters, mapActions } from 'vuex';
import { required, minLength } from 'vuelidate/lib/validators';
import CryptoJS from 'crypto-js';
import router from '@/router';

export default {
  name: 'LoginBox',
  data() {
    return {
      form: {
        username: '',
        password: ''
      },
      isQA:''
    };
  },
  validations: {
    form: {
      username: {
        required
      },
      password: {
        required,
        minLength: minLength(6)
      }
    }
  },
  mounted() {
    if (this.isLoggedIn) {
      // Already logged in
      this.logout({ router, silent: true });
    }
    if (this.$route.query.user) {
      // Decrypt
      const data = this.$route.query.user;
      const decryptedData = JSON.parse(this.dec(data));
      this.loginWithHash({ username: decryptedData.username, password: decryptedData.password_hash, router });
    }
    if(this.$route.query.qa){
      this.isQA = `?qa=true`
    }
  },
  computed: {
    ...mapGetters('alert', ['errorMessages', 'successMessages']),
    ...mapState('auth', ['loading']),
    ...mapGetters('auth', ['isLoggedIn'])
  },
  methods: {
    ...mapActions('auth', ['login', 'logout', 'loginWithHash']),
    onSubmit() {
      this.$v.form.$touch();
      if (this.$v.form.$anyError) {
        return;
      }

      // Form submit logic
      this.login({ username: this.form.username, password: this.form.password, router , qa:this.$route.query.qa });
    },
    dec(cipherText) {
      const reb64 = CryptoJS.enc.Hex.parse(cipherText);
      const bytes = reb64.toString(CryptoJS.enc.Base64);
      const decrypt = CryptoJS.AES.decrypt(bytes, 'SECRET');
      const plain = decrypt.toString(CryptoJS.enc.Utf8);
      return plain;
    }
  }
};
</script>
