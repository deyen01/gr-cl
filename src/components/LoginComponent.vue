<template>
  <div class="container m-4">
    <h4>Вход</h4>
    <form>
      <div class="mb-3">
        <label for="email" class="form-label">Email адрес</label>
        <input type="email" class="form-control" id="email" name="email" v-model="email" v-on:keyup.enter="login">
      </div>
      <div class="mb-3">
        <label for="password" class="form-label">Пароль</label>
        <input type="password" class="form-control" id="password" name="password" v-model="password" v-on:keyup.enter="login">
      </div>
      <button type="button" class="btn btn-primary" v-on:click="login">Войти</button>
    </form>
  </div>
</template>

<script>
export default {
  name: 'LoginComponent',
  props: {},
  data() {
    return {
      email: '',
      password: ''
    }
  },
  methods: {
    async login() {
      await this.$parent.load_csrf();
      const formData = new FormData();
      formData.append('email', this.email);
      formData.append('password', this.password);
      //formData.append('_token', this.$parent.csrf);
      fetch(this.$parent.api_url + '/login', {
        credentials: 'include',
        method: 'POST',
        mode: 'cors',
        headers: {
          'Accept': 'application/json',
          'X-XSRF-TOKEN': this.$parent.csrf,
          'Origin': this.$parent.origin,
        },
        redirect: "follow",
        body: formData
      })
      .then((response) => {
        if (response.ok) {
          this.$parent.login_show = false;
          this.$parent.user_show = true;
        }
      })
      .catch((error) => {
        console.error('Error:', error);
      });
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>