<template>
  <form @submit.prevent="register">
    <fieldset>
      <legend>
        Register
      </legend>
      <div v-if="errors.length" class="errors">
        <b>Please correct the following error(s):</b>
        <ul>
          <!-- eslint-disable-next-line -->
          <li v-for="error in errors">{{ error }}</li>
        </ul>
      </div>
      <div>
        <label for="username">Username</label>
        <input
          id="username"
          name="username"
          required
          autofocus
          v-model="user.username"
          type="text"
          pattern="[0-9A-Za-z][0-9A-Za-z ]{3,15}"
          placeholder=""
        />
        <span></span>
        <span class="warning">4 to 15 letters, numbers or spaces</span>
      </div>
      <div>
        <label for="password">Password</label>
        <input
          id="password"
          name="password"
          required
          v-model="user.password"
          type="password"
          pattern=".{4,15}"
          placeholder=""
        />
        <span></span>
        <span class="warning">4 to 15 characters</span>
      </div>
      <div>
        <label for="confirm-password">Confirm Password</label>
        <input
          id="confirm-password"
          name="confirm-password"
          required
          v-model="confirmPassword"
          type="password"
          :pattern="user.password"
          placeholder=""
        />
        <span></span>
        <span class="warning">Match password</span>
      </div>
      <div class="">
        <input
          v-if="!loading"
          class="button"
          type="submit"
          value="Register"
          :disabled="!isValid"
        />
        <input
          v-if="loading"
          class="button"
          type="submit"
          value="in progress..."
          disabled
        />
      </div>
    </fieldset>
  </form>
</template>

<script>
import { mapState } from 'vuex';

export default {
  name: 'Register',
  data: function() {
    return {
      errors: [],
      user: {
        username: '',
        password: ''
      },
      confirmPassword: ''
    };
  },
  computed: {
    isValid: function() {
      return this.validUsername() && this.validPassword() && this.matchingPasswords();
    },
    ...mapState('users', { loading: 'isCreatePending' })
  },
  methods: {
    register: function(evt) {
      if (this.validForm()) {
        console.log("submitting form");
        // create user instance
        const { User } = this.$FeathersVuex;
        const user = new User(this.user);
        user.save()
          .then((user) => {
            console.log({user:user});
            this.$router.push('/login');
          });
      }
      evt.preventDefault();
    },
    validForm(){
      // form has html5 validation
      // do some js validation as backup
      let err = this.errors = [];
      if (!this.user.username) {
        err.push('Username is required.');
      } else if (!this.validUsername()) {
        err.push('Username must have 4 to 15 letters and/or numbers.');
      }
      if (!this.user.password) {
        err.push('Password is required.');
      } else if (!this.validPassword()) {
        err.push('Password must have 4 to 15 characters.');
      }
      if (!this.matchingPasswords()) {
        err.push('Both passwords must match.');
      }
      return err.length === 0;
    },
    validUsername: function() {
      var re = /^[0-9A-Za-z][0-9A-Za-z ]{3,15}$/;
      return re.test(this.user.username);
    },
    validPassword: function() {
      var re = /^.{4,15}$/;
      return re.test(this.user.password);
    },
    matchingPasswords: function() {
      return this.user.password === this.confirmPassword;
    }
  }
};
</script>

<style scoped>
fieldset {
  border: 1px solid #ccc;
  border-radius: 10px;
  display: inline-block;
}
legend {
  margin:1em 0;
  font-size: 1.1rem;
}
label {
  display:flex;
}
input {
  margin-bottom:1em;
}
input:invalid + span:after {
  content: '✖';
  color: #f00;
  padding-left: 5px;
}
input:valid + span:after {
  content: '✓';
  color: #26b72b;
  padding-left: 5px;
}
.warning {
  margin-left: 5px;
  font-size: .65rem;
  color: #e67d09;
}
.errors {
  font-size: .85rem;
  color: #e67d09;
}
ul {
  margin: 0 -1em 1em 0;
}
</style>