<template>
  <div class="container m-4" v-show="!payment_show">
    <h4>Пользователи</h4>
    <button type="button" class="m-2 btn btn-primary btn-sm" v-on:click="load_users">Обновить список</button>
    <table class="table table-striped table-hover"><thead><tr><th>ID</th><th>Имя</th><th>E-mail</th><th>Телефон</th><th>Новый пароль (оставьте пустым или не меньше 8 симв.)</th><th>Действия</th></tr></thead>
      <tbody>
        <tr>
          <td>0</td>
          <td><input type="text" class="form-control" v-model="newuser.name"></td>
          <td><input type="text" class="form-control" v-model="newuser.email"></td>
          <td><input type="text" class="form-control" v-model="newuser.tel"></td>
          <td><input type="text" class="form-control" v-model="newuser.password"></td>
          <td>
            <button type="button" class="btn btn-warning btn-sm" v-on:click="user_store">Создать</button>
          </td>
        </tr>
        <tr v-for="user in users" v-bind:key="user.id">
          <td>{{ user.id }}</td>
          <td><input type="text" class="form-control" v-model="user.name"></td>
          <td><input type="text" class="form-control" v-model="user.email"></td>
          <td><input type="text" class="form-control" v-model="user.tel"></td>
          <td><input type="text" class="form-control" v-model="user.password"></td>
          <td>
            <button type="button" class="m-2 btn btn-warning btn-sm" v-on:click="user_update(user)">Изменить</button>
            <button type="button" class="m-2 btn btn-danger btn-sm" v-on:click="user_delete(user)">Удалить</button>
            <button type="button" class="m-2 btn btn-info btn-sm" v-on:click="user_payments(user)">Платежи</button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
  <PaymentComponent v-if="payment_show" v-bind:user="selected_user"></PaymentComponent>
</template>

<script>
import PaymentComponent from './PaymentComponent.vue'
export default {
  name: 'UserComponent',
  components: {
    PaymentComponent,
  },
  data() {
    return {
      users: [],
      newuser: {
        name: '',
        email: '',
        tel: ''
      },
      payment_show: false,
      selected_user: 0
    }
  },
  methods: {
    load_users() {
      fetch(this.$parent.api_url + '/api/users', {
        credentials: 'include',
        method: 'GET',
        headers: {
          'Accept': 'application/json',
          'X-XSRF-TOKEN': this.$parent.csrf,
          'Origin': this.$parent.origin,
        },
      })
      .then((response) => {
        return response.json();
      })
      .then((data) => {
        this.users = data.users;
        console.log(this.users);
      });
    },
    user_update(user) {
      let u = {
        name: user.name,
        email: user.email,
        tel: user.tel,
        password: user.password
      }
      fetch(this.$parent.api_url + '/api/users/' + user.id, {
        credentials: 'include',
        method: 'PATCH',
        mode: 'cors',
        headers: {
          'Accept': 'application/json',
          'Content-Type': 'application/json',
          'X-XSRF-TOKEN': this.$parent.csrf,
          'Origin': this.$parent.origin,
        },
        redirect: "follow",
        body: JSON.stringify(u)
      })
      .then((response) => {
        if (response.ok) {
          this.load_users();
        }
      })
      .catch((error) => {
        console.error('Error:', error);
      });
    },
    user_store() {
      let u = {
        name: this.newuser.name,
        email: this.newuser.email,
        tel: this.newuser.tel,
        password: this.newuser.password
      }
      fetch(this.$parent.api_url + '/api/users', {
        credentials: 'include',
        method: 'POST',
        mode: 'cors',
        headers: {
          'Accept': 'application/json',
          'Content-Type': 'application/json',
          'X-XSRF-TOKEN': this.$parent.csrf,
          'Origin': this.$parent.origin,
        },
        redirect: "follow",
        body: JSON.stringify(u)
      })
      .then((response) => {
        if (response.ok) {
          this.newuser.name = '';
          this.newuser.email = '';
          this.newuser.tel = '';
          this.newuser.password = '';
          this.load_users();
        }
      })
      .catch((error) => {
        console.error('Error:', error);
      });
    },
    user_delete(user) {
      fetch(this.$parent.api_url + '/api/users/' + user.id, {
        credentials: 'include',
        method: 'DELETE',
        mode: 'cors',
        headers: {
          'Accept': 'application/json',
          'Content-Type': 'application/json',
          'X-XSRF-TOKEN': this.$parent.csrf,
          'Origin': this.$parent.origin,
        },
        redirect: "follow",
      })
      .then((response) => {
        if (response.ok) {
          this.load_users();
        }
      })
      .catch((error) => {
        console.error('Error:', error);
      });
    },
    user_payments(user) {
      this.selected_user = user;
      this.payment_show = true;
    }
  },
  created() {
    this.load_users();
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>