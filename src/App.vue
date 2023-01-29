<template>
<nav v-if="menu_show" class="navbar navbar-expand-lg navbar-light bg-light">
  <div class="container-fluid">
    <a class="navbar-brand" href="/">GR</a>
    <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarSupportedContent" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle navigation">
      <span class="navbar-toggler-icon"></span>
    </button>
    <div class="collapse navbar-collapse" id="navbarSupportedContent">
      <ul class="navbar-nav me-auto mb-2 mb-lg-0">
        <li class="nav-item">
          <a class="nav-link active" aria-current="page" href="#" v-on:click="user_show = !user_show">Пользователи</a>
        </li>
      </ul>
    </div>
  </div>
</nav>
<div v-if="cu.id">
  <h3>Привет, {{ cu.name }}</h3>
  <a v-on:click="logout">Выход</a>
</div>
<LoginComponent v-if="login_show"></LoginComponent>
<UserComponent v-if="user_show"></UserComponent>
</template>

<script>
import LoginComponent from './components/LoginComponent.vue'
import UserComponent from './components/UserComponent.vue'

export default {
  name: 'App',
  components: {
    LoginComponent,
    UserComponent
  },
  data() {
    return {
      api_url: "http://gr-api.deyen.net",
      origin: 'http://gr-cl.deyen.net',
      csrf: '',
      cu: {}, // current user
      menu_show: true,
      login_show: true,
      user_show: false,
    }
  },
  methods: {
    logout() {
      const formData = new FormData();
      //formData.append('_token', this.csrf);
      fetch(this.api_url + '/logout', {
        credentials: 'include',
        method: 'POST',
        mode: 'cors',
        headers: {
          'Accept': 'application/json',
          'X-XSRF-TOKEN': this.csrf,
          'Origin': this.origin,
        },
        redirect: "follow",
        body: formData
      })
      .then((response) => {
        if (response.ok) {
          this.login_show = true;
          this.user_show = false;
        }
      })
      .catch((error) => {
        console.error('Error:', error);
      });
    },
    load_csrf() {
      fetch(this.api_url + '/sanctum/csrf-cookie', {
        credentials: 'include',
        method: 'GET',
        headers: {
          'Accept': 'application/json',
          'Origin': this.origin
        },
      })
      .then((response) => {
        console.log(response.ok);
        this.csrf = this.getCookie('XSRF-TOKEN');
        console.log('csrf is ' + this.csrf);
      });
    },
    check_auth() {
      fetch(this.api_url + '/api/user', {
        credentials: 'include',
        method: 'GET',
        headers: {
          'Accept': 'application/json',
          'X-XSRF-TOKEN': this.csrf,
          'Origin': this.origin
        },
      })
      .then((response) => {
        return response.json();
      })
      .then((data) => {
        if (data.id) {
          this.cu = data;
          this.login_show = false;
          this.user_show = true;
        } else {
          this.login_show = true;
          this.user_show = false;
        }
      });
    },
    getCookie(cname) {
      let name = cname + "=";
      let decodedCookie = decodeURIComponent(document.cookie);
      let ca = decodedCookie.split(';');
      for(let i = 0; i <ca.length; i++) {
        let c = ca[i];
        while (c.charAt(0) == ' ') {
          c = c.substring(1);
        }
        if (c.indexOf(name) == 0) {
          return c.substring(name.length, c.length);
        }
      }
      return "";
    }
  },
  async created() {
    this.csrf = localStorage.csrf;
    await this.load_csrf();
    await this.check_auth();
  },
  watch: {
    csrf: function () {
      localStorage.csrf = this.csrf;
    }
  }
}
</script>

<style>
#app {}
</style>
