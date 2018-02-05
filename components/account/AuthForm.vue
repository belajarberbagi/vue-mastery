<template lang="pug">
form.form(v-on:submit.prevent="submit")
  h2.form-title {{ title }}
  .form-group
    label.label Email
    input.input(v-bind:class="{ '-is-error': invalidEmail }" type="email" placeholder="Account Email" v-model="email" autocomplete="email")
    span.help-text.-is-error(v-if="invalidEmail" v-cloak) This email is invalid

  .form-group(v-if="rememberPassword" v-cloak)
    label.label Password
    input.input(v-bind:class="{ '-is-error': invalidPassword }" type="password" placeholder="Password" v-model="password")
    span.help-text.-is-error(v-if="invalidPassword" v-cloak) This password is invalid

  .form-group(v-if="isNew" v-cloak)
    label.checkbox
      input(type="checkbox" name="terms" v-model="terms" @click="checkDisabled")
      span I accept the terms and conditions
  
  .form-error
    .error(v-if="formError.length > 0" v-text="formError" v-cloak)

  .form-actions(@click="checkDisabled")
    button.button.primary.-small(type="submit" :disabled="actionDisabled") {{ label }}
    .control-group(v-if="rememberPassword" v-cloak)
      GoogleButton.button.inverted.-small(:label="label + ' with Google'" :disabled="actionDisabled")
      GithubButton.button.inverted.-small(:label="label + ' with Github'" :disabled="actionDisabled")

  .form-footer.-spaced
    .control-group(v-if="isNew" v-cloak)
      button.button.link(type="button" @click="switchForm") I already have an account
    .control-group(v-else)
      button.button.link(type="button" @click="switchForm" v-if="rememberPassword" v-cloak) Signup
      button.button.link(type="button" @click="retrievePassword" v-if="rememberPassword" v-cloak) Forgot your password?
      button.button.link(type="button" @click="switchForm" v-if="!rememberPassword" v-cloak) Or Sign in

</template>

<script>
import GoogleButton from '~/components/account/3rd-party/GoogleButton.vue'
import GithubButton from '~/components/account/3rd-party/GithubButton.vue'

export default {
  name: 'LoginForm',
  components: {
    GoogleButton,
    GithubButton
  },
  props: {
    newAccount: {
      default: false
    }
  },
  data () {
    return {
      email: '',
      password: '',
      formError: '',
      isNew: this.newAccount,
      terms: !this.newAccount,
      rememberPassword: true
    }
  },
  computed: {
    title () {
      let t = this.rememberPassword ? 'Welcome back!' : 'Retrieve your password'
      if (this.isNew) t = 'Let\'s Get You Signed Up.'
      return t
    },
    label () {
      let l = this.rememberPassword ? 'Log In' : 'Reset'
      if (this.isNew) l = 'Sign Up'
      return l
    },
    invalidEmail () {
      return !this.email.includes('@') && this.email !== ''
    },
    invalidPassword () {
      return !this.password.length > 6
    },
    actionDisabled () {
      return this.isNew ? !this.terms : false
    }
  },
  methods: {
    switchForm () {
      this.rememberPassword = true
      this.isNew = !this.isNew
      this.terms = false
    },
    retrievePassword () {
      this.rememberPassword = false
    },
    checkDisabled () {
      this.formError = ''
      if (!this.terms) this.formError = 'Please read and accept the terms and conditions'
    },
    checkForm () {
      let invalid = false
      if (this.rememberPassword) invalid = this.invalidPassword
      if (this.invalidEmail) invalid = true
      return invalid
    },
    submit () {
      // Check if the form is valid and display errors
      this.formError = ''
      if (this.checkForm()) return false

      // Define which action we do according to the state
      let action = 'userCreate'
      if (!this.isNew) {
        action = this.rememberPassword ? 'userLogin' : 'userRetrievePassword'
      }

      // Dispach information to the store
      this.$store.dispatch(action, {
        email: this.email,
        password: this.password
      })
        .then(() => {
          if (this.isNew) this.$router.push('/account')
        })
        .catch((error) => {
          console.log(error)
          this.formError = error.message
        })
    }
  }
}
</script>

<style lang="stylus" scoped></style>