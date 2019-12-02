<template>
  <form @submit.prevent="doLogin()" class="form-login">
    <div class="card">
      <div class="card-header text-center">
        <h1 class="mb-0">Expenses</h1>
      </div>
      <div class="card-body">
        <div class="form-group">
          <input type="email" v-model="email" class="form-control" placeholder="E-mail" required />
        </div>
        <div class="form-group">
          <input
            type="password"
            v-model="password"
            class="form-control"
            placeholder="Senha"
            required
          />
        </div>
        <button class="btn btn-primary w-100" :disabled="loading">
          <template v-if="loading">
            Entrando...
            <i class="fa fa-spinner fa-spin"></i>
          </template>

          <template v-else>
            Entrar
            <i class="fa fa-sign-in-alt"></i>
          </template>
        </button>
      </div>
    </div>
  </form>
</template>

<script>
export default {
  name: 'login',
  data: () => {
    return {
      loading: false,
      email: '',
      password: ''
    }
  },
  mounted () {
    if (process.env.NODE_ENV === 'development') {
      this.email = process.env.VUE_APP_LOGIN_EMAIL
      this.password = process.env.VUE_APP_LOGIN_PASS
    }
  },
  methods: {
    async doLogin () {
      this.loading = true
      const { email, password } = this
      try {
        const res = await this.$firebase
          .auth().signInWithEmailAndPassword(email, password)
        window.uid = res.user.uid

        this.$router.push({ name: 'home' })
      } catch (err) {
        let message = ''

        switch (err.code) {
          case 'auth/user-not-found':
            message = 'Não foi possível localizar o usuário.'
            break

          case 'auth/wrong-password':
            message = 'Senha Inválida'
            break

          default:
            message = 'Não foi possível fazer o login, tente novamente'
        }

        this.$root.$emit('Notification::show', {
          message,
          type: 'danger'
        })
      }
      this.loading = false
    }
  },
  beforeRouteEnter (to, from, next) {
    next(vm => {
      if (window.uid) {
        vm.$router.push({ name: 'home' })
      }
    })
  }
}
</script>

<style lang="scss" scoped>
.form-login {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
}

h1 {
  font-size: 18pt;
}

.card {
  width: 30%;
  color: var(--darker);
}
</style>
