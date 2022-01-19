<template>
  <v-data-table :headers="headers" :items="logins" class="elevation-1">
    <template v-slot:top>
      <v-toolbar flat>
        <v-toolbar-title>Login Information</v-toolbar-title>
        <v-divider class="mx-4" inset vertical></v-divider>
        <v-spacer></v-spacer>
        <v-dialog v-model="dialog" max-width="500px">
          <template v-slot:activator="{ on, attrs }">
            <v-btn color="primary" dark class="mb-2" v-bind="attrs" v-on="on">
              New Login Information
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
                        :items="customerIDs"
                        filled
                        label="Customer ID"
                        :rules="[rules.required]"
                      ></v-select>
                    </v-col>
                    <v-col cols="12">
                      <v-text-field
                        v-model="editedItem.customer_username"
                        label="Customer UserName"
                        :rules="[rules.required, rules.counter]"
                      ></v-text-field>
                    </v-col>
                      <v-col cols="12">
                      <v-text-field
                        v-model="editedItem.customer_password"
                        label="Customer Password"
                        :rules="[rules.required, rules.counter]"
                      ></v-text-field>
                    </v-col>
                      
                     <v-col cols="12">
                       <v-menu>
                       <template v-slot:activator="{on}">
                        <v-text-field
                          v-model="editedItem.signup_date"
                          v-on="on"
                          label="Sign Up Date"
                          :rules="[v => !!v || 'Required']"
                        ></v-text-field>
                      </template>
                        <v-date-picker 
                        v-model="editedItem.signup_date"
                        
                      ></v-date-picker>
                       </v-menu>
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
              <v-btn color="blue darken-1" text @click="deleteLoginBackend"
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
      { text: "Username", sortable: false, value: "customer_username" },
      { text: "Password", sortable: false, value: "customer_password" },
      { text: "Sign Up Date", sortable: false,value:"signup_date" },
      { text: "Actions", value: "actions", sortable: false },
      
    ],
    logins: [],
    editedIndex: -1,
    editedItem: {
      customer_id: 0,
      customer_username: "",
      customer_password:"",
      signup_date:"",
    },
    defaultItem: {
        customer_id: 0,
        customer_username: "",
        customer_password:"",
        signup_date:"",
      
    },
    customerIDs : []
  }),

  computed: {
    formTitle() {
      return this.editedIndex === -1 ? "New Login Information" : "Edit Login Information";
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
      this.getLogins();
      this.getCustomerIDs();
    },

    getCustomerIDs(){
      axios.get("http://localhost:8080/api/transaction/CustomerIDs").then((response) => {
        this.customerIDs = response.data;
      });
    },

    getLogins() {
      axios
        .get("http://localhost:8080/api/login_information")
        .then((response) => {
          this.logins = response.data;
        });
    },

    deleteLoginBackend() {
      axios
        .delete(
          `http://localhost:8080/api/login_information?customer_id=${this.editedItem.customer_id}`
        )
        .then(() => {
          this.deleteItemConfirm();
        });
    },

    editItem(item) {
      this.editedIndex = this.logins.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialog = true;
    },

    deleteItem(item) {
      this.editedIndex = this.logins.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialogDelete = true;
    },

    deleteItemConfirm() {
      this.logins.splice(this.editedIndex, 1);
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
          `http://localhost:8080/api/login_information?&customer_id=${this.editedItem.customer_id}&customer_username=${this.editedItem.customer_username}&customer_password=${this.editedItem.customer_password}&signup_date=${this.editedItem.signup_date}`
        )
        .then(() => {
          //this.categories.push(response.body);
          this.getLogins();
          this.close();
        });
    },

   /* updateBackend(){
      axios
        .put(
          `http://localhost:8080/api/instrument_category?&category_id=${this.editedItem.category_id}&category_name=${this.editedItem.category_name}`
        )
        .then(() => {
          //this.categories.push(response.body);
          //this.getCategories();
          Object.assign(this.categories[this.editedIndex], this.editedItem);
          this.close();
        });
    }*/
  },
};
</script>