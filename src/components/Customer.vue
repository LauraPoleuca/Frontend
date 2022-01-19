<template>
  <v-data-table :headers="headers" :items="customers" class="elevation-1">
    <template v-slot:top>
      <v-toolbar flat>
        <v-toolbar-title>Customer</v-toolbar-title>
        <v-divider class="mx-4" inset vertical></v-divider>
        <v-spacer></v-spacer>
        <v-dialog v-model="dialog" max-width="500px">
          <template v-slot:activator="{ on, attrs }">
            <v-btn color="primary" dark class="mb-2" v-bind="attrs" v-on="on">
              New Customer
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
                      <v-text-field
                        v-model="editedItem.customer_firstname"
                        label="Customer FirstName"
                        :rules="[rules.required, rules.counter,rules.notDigit]">
                      </v-text-field>
                    </v-col>
                    <v-col cols="12">
                      <v-text-field
                        v-model="editedItem.customer_lastname"
                        label="Customer LastName"
                        :rules="[rules.required, rules.counter,rules.notDigit]"
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12">
                      <v-text-field 
                        v-model="editedItem.customer_email"
                        label="Customer Email"
                        :rules="[rules.required, v => !v || /^\w+([.-]?\w+)*@\w+([.-]?\w+)*(\.\w{2,3})+$/.test(v) || 'E-mail must be valid']"
                        
                      ></v-text-field>
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
              <v-btn color="blue darken-1" text @click="deleteCustomerBackend"
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
      counter: (value) => value.length <= 30 || "Max 30 characters",
      notDigit: (value) => /\d/.test(value) === false || "No digits allowed",
    },
    isFormValid:false,
    dialog: false,
    dialogDelete: false,
    headers: [
      {
        text: "Customer ID",
        align: "start",
        sortable: false,
        value: "customer_id",
      },
      {
        text: "FirstName",
        sortable: false,
        value: "customer_firstname",
      },
      {
        text: "LastName",
        sortable: false,
        value: "customer_lastname",
      },
      { text: "Email", sortable: false, value: "customer_email" },
      { text: "Actions", value: "actions", sortable: false },
    ],
    customers: [],
    editedIndex: -1,
    editedItem: {
      customer_id: 0,
      customer_firstname: "",
      customer_lastname: "",
      customer_email: "",
    },
    defaultItem: {
      customer_id: 0,
      customer_firstname: "",
      customer_lastname: "",
      customer_email: "",
    },
  }),

  computed: {
    formTitle() {
      return this.editedIndex === -1 ? "New Customer" : "Edit Customer";
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
      this.getCustomers();
    },

    getCustomers() {
      axios.get("http://localhost:8080/api/customer").then((response) => {
        this.customers = response.data;
      });
    },

    deleteCustomerBackend() {
      axios
        .delete(
          `http://localhost:8080/api/customer?customer_id=${this.editedItem.customer_id}`
        )
        .then(() => {
          this.deleteItemConfirm();
        });
    },

    editItem(item) {
      this.editedIndex = this.customers.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialog = true;
    },

    deleteItem(item) {
      this.editedIndex = this.customers.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialogDelete = true;
    },

    deleteItemConfirm() {
      this.customers.splice(this.editedIndex, 1);
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
          `http://localhost:8080/api/customer?&customer_firstname=${this.editedItem.customer_firstname}&customer_lastname=${this.editedItem.customer_lastname}&customer_email=${this.editedItem.customer_email}`
        )
        .then(() => {
          //this.categories.push(response.body);
          this.getCustomers();
          this.close();
        });
    },
  },
};
</script>