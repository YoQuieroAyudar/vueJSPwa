<template>
  <div class="login-area-wrapper">
    <div class="share-button">
      <button :class="$i18n.locale() == 'ar' ? 'btn btn-success btn-xs pull-left' : 'btn btn-success btn-xs pull-right'" type='button' @click="$store.commit('setShowShare', true)">{{$t('Share')}}</button>
    </div>

    <h1 class="page-top-title"><span v-if="$store.getters.getLoginAsUser">{{$t('Login as User')}}</span><span v-else>{{$t('Login as Establishment')}}</span></h1>
    <div class="user-login" v-if="$store.getters.getLoginAsUser">

      <form class="form">
        <div dir="ltr" class="input-group">
          <span class="input-group-addon" :title="$t('Country')" id="country-addon1"> {{$t('Donation Destination')}}</span>
          <select class="form-control" aria-describedby="nationality-addon1" @change="updateAPI" v-model="country" >
            <option v-for="(ctry, i) in $store.getters.getTopCountries" :selected="i == 1" :value="ctry.code">{{ctry.name}}</option>
          </select>
        </div>
        <div dir="ltr" class="input-group" :title="$t('Email')">
          <span class="input-group-addon" id="email-addon1"> <i class="fa fa-envelope fa-fw" aria-hidden="true"></i> </span>
          <input name="mail" class="form-control" v-model="login.mail" @input="updateEmail" aria-describedby="email-addon1" type="email" :placeholder="$t('Email')" :value="login.email" />
        </div>

        <div dir="ltr" class="input-group" :title="$t('Password')">
          <span class="input-group-addon" id="password-addon1"> <i class="fa fa-lock fa-fw" aria-hidden="true"></i> </span>
          <input name="password" class="form-control" v-model="login.password"  @input="updatePassword" aria-describedby="password-addon1" type="password" :placeholder="$t('Password')" :value="login.password" />
        </div>

        {{ $t("If you don't have an account yet") }} <a class="" @click="goToSignupPage" > {{ $t('Sign up here') }}</a>

        <p class = "tpbutton btn-toolbar text-center">
          <button class="btn navbar-btn btn-primary login-btn" @click="loginUser" > <i class="fa fa-paper-plane" aria-hidden="true"></i> {{ $t('Login') }}</button>
          <button class="btn navbar-btn btn-primary login-btn" @click="$store.commit('setShowSocialLogin', true)" type="button" name="button"><i class="fa fa-external-link fa-fw"></i> {{$t('Social Login')}}</button>
        </p>

        {{ $t('Remember me') }} <input name="remember_me" v-model="rememberMe" @click="setRememberMe" :checked="rememberMe" aria-describedby="password-addon1" type="checkbox" :value="rememberMe" />
      </form>

      <video-frame v-if="$store.getters.getShowVideo"></video-frame>

    </div>

    <div class="POS-login" v-else>
      <login-pos-form></login-pos-form>
    </div>
    <button class="btn btn-default btn-xs btn-block" @click="gotToLoginPOS">
      <span v-if="!$store.getters.getLoginAsUser">{{$t('Login as User')}}</span><span v-else>{{$t('Login as Establishment')}}</span>
    </button>
  </div>
</template>

<style scoped>
h1, h2, h3, h4, h5 {
  font-weight: bold;
}
h1 {
  font-size: 1.45em;
}
h2{
  font-size: 1.3em;
}
h3{
  font-size: 1.15em;
}
h4 {
  font-size: 1em;
}
h5 {
  font-size: 0.95em;
}

a {
  cursor: pointer;
}
.login-btn {
  margin-top: .1em;
  margin-bottom: .2em;
  width: 48%;
}
.fb-login-button {
}
.login-area-wrapper {
  margin: auto;
}
.input-group {
  margin-top: .1em;
}
.btn-xs {
  margin-top: .2em;
}
.share-button {
  position: absolute;
  left: 0;
  right: 1.7em;
  margin-top: 0;
}
.pull-left {
  left: 0;
  margin-left: 2em;
}
.page-top-title {
  width: 75%;
  border: 1px solid #EFE;
  padding: .1em;
}
</style>

<script>
import VideoFrame from './Vids.vue'
import LoginPOS from './LoginPOS.vue'

export default {

  data () {
    return {
      login: {
        mail: '',
        password: ''
      },
      rememberMe: localStorage.getItem('rememberMe'),
      facebookLogin: false,
      // country: {name: 'Spain', db: 'mhs', code: 'ES'},
      asUser: true
    }
  },
  computed: {
    country: {
      get () {
        return this.$store.getters.getSelectedCountry.toUpperCase()
      },
      set (newValue) {
        console.log('new value for country')
        console.log(newValue)
        this.$store.commit('setSelectedCountry', newValue.toUpperCase())
      }
    }
  },
  methods: {
    gotToLoginPOS () {
      // this.$store.commit('resetMessages')
      this.$store.commit('setLoginAsUser', !this.asUser)
      // this.$store.commit('setCurrentState', 'login')
      this.asUser = !this.asUser
    },
    setRememberMe () {
      console.log('rememberMe: ' + this.rememberMe)
      localStorage.setItem('rememberMe', this.rememberMe)
      console.log('rememberMe: ' + localStorage.getItem('rememberMe'))
    },
    updateAPI () {
      this.$store.commit('setAPI', this.country.db)
      localStorage.setItem('country', JSON.stringify(this.country))
      localStorage.setItem('country_code', this.country.code)
      localStorage.setItem('country_db', this.country.db)
    },
    goToSignupPage (e) {
      e.preventDefault()
      this.$store.commit('resetMessages')
      // this.$store.commit('setCurrentPage', 'signup')
      this.$events.emit('goToPageEvent', 'signup')
      this.$store.commit('setCurrentState', '')
    },
    updateEmail (e) {
      this.$store.commit('updateEmail', this.login.mail)
    },
    updatePassword (e) {
      this.$store.commit('updatePassword', this.login.password)
    },
    setMessage (response) {
      if (!response) {
        return
      }

      var vm = this
      setTimeout(() => { vm.$store.commit('resetMessages') }, 5000)

      if (response.errors) {
        this.$store.commit('setErrors', response.errors)
      } else if (response.error) {
        this.$store.commit('setError', response.error)
      } else if (response.warning) {
        this.$store.commit('setWarning', response.warning)
      } else if (response.info) {
        this.$store.commit('setInfo', response.info)
      } else if (response.success) {
        this.$store.commit('setSuccess', response.success)
      }

      this.$store.commit('setLoading', false)
    },
    setUserToken (token) {

    },
    loginUser (e) {
      e.preventDefault()
      console.log('COUNTRY: ' + this.country)
      this.$events.emit('sendLoginEvent', this.login)
    }
  },
  components: {
    'login-pos-form': LoginPOS,
    'video-frame': VideoFrame
  }

}
</script>
