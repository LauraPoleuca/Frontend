<template>
  <v-data-table :headers="headers" :items="transactions" class="elevation-1">
    <template v-slot:top>
      <v-toolbar flat>
        <v-toolbar-title>Transaction</v-toolbar-title>
        <v-divider class="mx-4" inset vertical></v-divider>
        <v-spacer></v-spacer>
        <v-dialog v-model="dialog" max-width="500px">
          <template v-slot:activator="{ on, attrs }">
            <v-btn color="primary" dark class="mb-2" v-bind="attrs" v-on="on">
              New Transaction
            </v-btn>
          </template>
          <v-card>
            <v-card-title>
              <span class="text-h5">{{ formTitle }}</span>
            </v-card-title>

            <v-card-text>
              <v-container>
                <v-form v-model="isFormValid">
                  <v-row>
                    <v-col cols="12">
                      <v-select v-model="editedItem.customer_id"
                        :items="CustomerIDs"
                        filled
                        label="Customer ID"
                        :rules="[rules.required]"
                      ></v-select>
                    </v-col>
                    <v-col cols="12">
                      <v-text-field
                        v-model="editedItem.total_price"
                        label="Total Price"
                        type="number"
                        :rules="[rules.required,rules.greaterThanZero]"
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12">
                      <v-text-field
                        v-model="editedItem.method_of_payment"
                        label="Method of Payment"
                        :rules="[rules.required, rules.counter]"
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12">
                        <v-date-picker label="Transaction Date"
                        v-model="editedItem.transaction_date"
                        :rules="[rules.required]"
                      ></v-date-picker>
                    </v-col>
                  </v-row>
                </v-form>
              </v-container>
            </v-card-text>

            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" text @click="close"> Cancel </v-btn>
              <v-btn color="blue darken-1" text @click="save" :disabled="!isFormValid"> Save </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
        <v-dialog v-model="dialogDelete" max-width="500px">
          <v-card>
            <v-card-title class="text-h5"
              >Are you sure you want to delete this item?</v-card-title
            >
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" text @click="closeDelete"
                >Cancel</v-btn
              >
              <v-btn color="blue darken-1" text @click="deleteTransactionBackend"
                >OK</v-btn
              >
              <v-spacer></v-spacer>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-toolbar>
    </template>
    <template v-slot:item.actions="{ item }">
      
      <v-icon small @click="deleteItem(item)"> mdi-delete </v-icon>
    </template>
    <template v-slot:no-data>
      <v-btn color="primary" @click="initialize"> Reset </v-btn>
    </template>
  </v-data-table>
</template>

<script>
import axios from "axios";
export default {
  data: () => ({
    rules: {
      required: (value) => !!value || "Required.",
      counter: (value) => value.length <= 20 || "Max 20 characters",
      greaterThanZero:(value)=> parseInt(value)>0 || "Quantity must be greater than zero",
    },
    isFormValid:false,
    dialog: false,
    dialogDelete: false,
    headers: [
      {
        text: "Transaction ID",
        align: "start",
        sortable: false,
        value: "transaction_id",
      },
      {
        text: "Customer ID",
        sortable: false,
        value: "customer_id",
      },
      {
        text: "Total Price",
        sortable: false,
        value: "total_price",
      },
      { text: "Method of Payment", sortable: false, value: "method_of_payment" },
      { text: "Transaction Date", sortable: false, value: "transaction_date" },
      { text: "Actions", value: "actions", sortable: false },
    ],
    transactions: [],
    CustomerIDs: [],
    editedIndex: -1,
    editedItem: {
      transaction_id: 0,
      customer_id:0,
      total_price:0,
      method_of_payment: "",
      transaction_date:"",
    },
    defaultItem: {
        transaction_id: 0,
        customer_id:0,
        total_price:0,
        method_of_payment: "",
        transaction_date:"",
     
    },
  }),

  computed: {
    formTitle() {
      return this.editedIndex === -1 ? "New Transaction" : "Edit Transaction";
    },
  },

  watch: {
    dialog(val) {
      val || this.close();
    },
    dialogDelete(val) {
      val || this.closeDelete();
    },
  },

  created() {
    this.initialize();
  },

  methods: {
    initialize() {
      this.getTransactions();
      this.getCustomerIDs();
    },

    getCustomerIDs(){
      axios.get("http://localhost:8080/api/transaction/CustomerIDs").then((response) => {
        this.CustomerIDs = response.data;
      });
    },

    getTransactions() {
      axios.get("http://localhost:8080/api/transaction").then((response) => {
        this.transactions = response.data;
      });
    },

    deleteTransactionBackend() {
      axios
        .delete(
          `http://localhost:8080/api/transaction?transaction_id=${this.editedItem.transaction_id}`
        )
        .then(() => {
          this.deleteItemConfirm();
        });
    },

    editItem(item) {
      this.editedIndex = this.transactions.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialog = true;
    },

    deleteItem(item) {
      this.editedIndex = this.transactions.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialogDelete = true;
    },

    deleteItemConfirm() {
      this.transactions.splice(this.editedIndex, 1);
      this.closeDelete();
    },

    close() {
      this.dialog = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      });
    },

    closeDelete() {
      this.dialogDelete = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      });
    },

    save() {
      if (this.editedIndex > -1) {
        //UPDATE
        this.updateBackend();
        //Object.assign(this.categories[this.editedIndex], this.editedItem);//TODO: add the this.close() method
      } else {
        //CREATE
        this.createBackend();
      }
    },
    createBackend() {
      axios
        .post(
          `http://localhost:8080/api/transaction?&customer_id=${this.editedItem.customer_id}&total_price=${this.editedItem.total_price}&method_of_payment=${this.editedItem.method_of_payment}&transaction_date=${this.editedItem.transaction_date}`
        )
        .then(() => {
          //this.categories.push(response.body);
          this.getTransactions();
          this.close();
        });
    },
  },
};
</script>