<template>
  <div class="container m-4">
    <button type="button" class="btn btn-primary btn-sm" v-on:click="$parent.payment_show  = false">Назад</button>
    <h4>Платежи пользователя {{ user.name }} ID {{ user.id }}</h4>
    <button type="button" class="m-2 btn btn-primary btn-sm" v-on:click="load_payments">Обновить список</button>
    <table class="table table-striped table-hover"><thead><tr><th>ID</th><th>получатель платежа</th><th>сумма платежа</th><th>назначение платежа</th><th>Действия</th></tr></thead>
      <tbody>
        <tr>
          <td>0</td>
          <td><input type="text" class="form-control" v-model="newpayment.payee"></td>
          <td><input type="number" class="form-control" v-model="newpayment.sum"></td>
          <td><input type="text" class="form-control" v-model="newpayment.purpose"></td>
          <td>
            <button type="button" class="btn btn-warning btn-sm" v-on:click="payment_store">Создать</button>
          </td>
        </tr>
        <tr v-for="payment in payments" v-bind:key="payment.id">
          <td>{{ payment.id }}</td>
          <td>{{ payment.payee }}</td>
          <td>{{ payment.sum }}</td>
          <td>{{ payment.purpose }}</td>
          <td></td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
export default {
  name: 'PaymentComponent',
  props: {
    user: Object
  },
  data() {
    return {
      payments: [],
      newpayment: {
        payee: '',
        sum: 0,
        purpose: ''
      }
    }
  },
  methods: {
    load_payments() {
      fetch(this.$parent.$parent.api_url + '/api/users/' + this.user.id + '/payments', {
        credentials: 'include',
        method: 'GET',
        headers: {
          'Accept': 'application/json',
          'X-XSRF-TOKEN': this.$parent.$parent.csrf,
          'Origin': this.$parent.$parent.origin,
        },
      })
      .then((response) => {
        return response.json();
      })
      .then((data) => {
        this.payments = data.payments;
      });
    },
    payment_store() {
      let p = {
        payee: this.newpayment.payee,
        sum: this.newpayment.sum,
        purpose: this.newpayment.purpose
      }
      fetch(this.$parent.$parent.api_url + '/api/users/' + this.user.id + '/payments', {
        credentials: 'include',
        method: 'POST',
        mode: 'cors',
        headers: {
          'Accept': 'application/json',
          'Content-Type': 'application/json',
          'X-XSRF-TOKEN': this.$parent.$parent.csrf,
          'Origin': this.$parent.$parent.origin,
        },
        redirect: "follow",
        body: JSON.stringify(p)
      })
      .then((response) => {
        if (response.ok) {
          this.newpayment.payee = '';
          this.newpayment.sum = '';
          this.newpayment.purpose = '';
          this.load_payments();
        }
      })
      .catch((error) => {
        console.error('Error:', error);
      });
    }
  },
  created() {
    this.load_payments();
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>