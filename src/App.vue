<template>
  <div id="app">
    <h1>Contas</h1>
    <table>
      <thead>
        <tr>
          <th>Agencia</th>
          <th>Conta</th>
          <th>Saldo</th>
          <th></th>
          <th></th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(account, index) in bankAccounts" v-bind:key="account.id">
          <td>{{ account.agency }}</td>
          <td>{{ account.number }}</td>
          <td>{{ formatCurrency(account.balance) }}</td>
          <td><button v-on:click="deposit(index)">Depositar</button></td>
          <td><button v-on:click="withdraw(index)">Sacar</button></td>
        </tr>
      </tbody>
    </table>
    <div>{{ errorMessage }}</div>
    <div class="deposit" v-if="showDeposit">
      <label>Deposito</label>
      <input type="number"  v-model="depositInput" step ="0.1" min="0"/>
      <button v-on:click="saveDeposit">Salvar</button>
    </div>
    <div v-if="showWithdraw">
      <label>Saque</label>
      <input type="number" v-model="withdrawInput" step ="0.1" min="0" />
      <button v-on:click="saveWithdraw">Salvar</button>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

const client = axios.create({
  baseURL: 'http://localhost:8000',
});

export default {
  name: 'App',
  data() {
    return {
      showDeposit: false,
      showWithdraw: false,
      bankAccounts: [],
      currentAccount: null,
      withdrawInput: '',
      withdrawDeposit: '',
      errorMessage: '',
    };
  },
  methods: {
    withdraw(accountId) {
      this.currentAccount = accountId;
      this.showDeposit = false;
      this.showWithdraw = !this.showWithdraw;
      this.withdrawInput = '';
    },
    deposit(accountId) {
      this.currentAccount = accountId;
      this.showDeposit = !this.showDeposit;
      this.currentAccount = accountId;
      this.depositInput = '';
    },
    formatCurrency(amount) {
      return new Intl.NumberFormat('pt-BR', { style: 'currency', currency: 'BRL' }).format(amount);
    },
    saveDeposit() {
      client
        .post('http://localhost:8000/api/bank-accounts/deposit', {
          typeId: 1,
          bankAccountId: this.bankAccounts[this.currentAccount].id,
          amount: parseFloat(this.depositInput),
        })
        .then(() => this.addBalance())
        .catch((error) => {
          this.errorMessage = error.response.message;
        });
    },
    saveWithdraw() {
      client
        .post('http://localhost:8000/api/bank-accounts/withdraw', {
          typeId: 2,
          bankAccountId: this.bankAccounts[this.currentAccount].id,
          amount: parseFloat(this.withdrawInput),
        })
        .then(() => this.subBalance())
        .catch((error) => {
          this.errorMessage = error.response.message;
        });
    },
    subBalance() {
      this.bankAccounts[this.currentAccount].balance -= parseFloat(this.withdrawInput);
    },
    addBalance() {
      this.bankAccounts[this.currentAccount].balance += parseFloat(this.depositInput);
    },
  },
  mounted() {
    client.get('/api/bank-accounts').then((data) => {
      this.bankAccounts = data.data;
    });
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  width: 100%;
}

table {
  width: 100%;
}
</style>
