<template>
    <v-container class="col-sm-4 offset-sm-4 col-md-4 offset-md-4 col-lg-4 offset-lg-4">
      <v-form>
        <v-text-field dark dense prepend-inner-icon="mdi-account" outlined type="text" placeholder="логин" v-model="login"></v-text-field>
        <v-text-field dark dense prepend-inner-icon="mdi-lock" outlined @keyup.enter="doLogin()" type="password" placeholder="пароль" v-model="password"></v-text-field>
        <v-btn block @click="doLogin()" variant="primary" class="btn btn-sm btn-primary btn-block">Вход</v-btn>
        <v-alert class="mt-5" type="error" v-if="alert" show dense>
          <small style="font-family: monospace">{{this.alertmessage}}</small>
        </v-alert>
      </v-form>
    </v-container>
</template>

<script> export default {
  data() {
    return {
      chartShow: false,
      alert: false,
      alertmessage: null,
      login: null,
      password: null,

      options: {
      },

      series: [{
        name: 'solved',
        data: []
      }]
    }

  },
  mounted(){

  },
  beforeMount(){
    //this.getstats()
  },
  methods: {
    async getClosestDate(){
      },
    async doLogin() {
      try {
        const res = await this.$api.post('login', {
          login: this.login,
          password: this.password
        }, { skipTokenCheck: true })
        this.$cookie.set('token', res.token)
        this.$emit('done')
        this.cal_obj = await this.$api.post('getclosestmonth',{})
        this.$storage.set('calYear', this.cal_obj.closest_year)
        this.$storage.set('calMonth', this.cal_obj.closest_month)
        this.$router.push('/Ktu')
      } catch (e) {
        this.alertmessage = e.message
        this.alert = true

        console.log(e.message)
      }
    }

  }

}
</script>
