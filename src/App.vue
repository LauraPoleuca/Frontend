<template>
  <v-app>
    <v-app-bar color="brown darken-2" elevation="2" dense app>
      <v-spacer></v-spacer>
      <v-toolbar-title class="flex text-center">
        <h2 class="font-weight-medium" style="color: #d7ccc8">Music Store</h2>
      </v-toolbar-title>
      <v-spacer></v-spacer>

      <v-avatar color="brown lighten-1">
        <span class="white--text text-h6">LP</span>
      </v-avatar>
    </v-app-bar>

    <v-bottom-navigation
      class="pa-md-15 mx-lg-auto"
      :value="value"
      color="brown"
      grow
    >
      <v-btn @click="btn1">
        <span>Category</span>
      </v-btn>

      <v-btn @click="btn2">
        <span>Instrument</span>
      </v-btn>

      <v-btn @click="btn3">
        <span>Transaction</span>
      </v-btn>

      <v-btn @click="btn4">
        <span>Transaction Detail</span>
      </v-btn>

      <v-btn @click="btn5">
        <span>Customer</span>
      </v-btn>

      <v-btn @click="btn6">
        <span>Login Information</span>
      </v-btn>
    </v-bottom-navigation>

    <!-- Sizes your content based upon application components -->
    <v-main class="pa-md-15 mx-lg-10" >
      <div v-if="to_display == 'Category'">
        <Category />
      </div>
      <div v-if="to_display == 'Instrument'">
        <Instrument/>
      </div>
      <div v-if="to_display == 'Transaction'">
        <Transaction />
      </div>
      <div v-if="to_display == 'Transaction Detail'">
        <Detail/>
      </div>
      <div v-if="to_display == 'Customer'">
        <Customer/>
      </div>
      <div v-if="to_display == 'Login Information'">
        <LoginInfo/>
      </div>
    </v-main>

    <v-footer app>
      <!-- -->
    </v-footer>
  </v-app>
</template>

<script>
//import HelloWorld from "./components/HelloWorld";
import Category from "./components/Category";
import Instrument from "./components/Instrument";
import Customer from "./components/Customer";
import Transaction from "./components/Transaction";
import Detail from "./components/Detail";
import LoginInfo from "./components/LoginInfo";

export default {
  name: "App",

  components: {
    Category,
    Instrument,
    Customer,
    Transaction,
    Detail,
    LoginInfo,
  },

  data: () => ({
    to_display: "",
  }),
  methods: {
    btn1: function () {
      this.to_display = "Category";
    },
    btn2: function () {
      this.to_display = "Instrument";
    },
    btn3: function () {
      this.to_display = "Transaction";
    },
    btn4: function () {
      this.to_display = "Transaction Detail";
    },
    btn5: function () {
      this.to_display = "Customer";
    },
    btn6: function () {
      this.to_display = "Login Information";
    },
  },
};
</script>

<style>
#app {
  background: url("images/i4.png") no-repeat center center fixed !important;
  background-size: cover;
}
</style>