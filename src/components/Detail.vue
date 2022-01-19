<template>
  <v-data-table :headers="headers" :items="details" class="elevation-1">
    <template v-slot:top>
      <v-toolbar flat>
        <v-toolbar-title>Transaction Detail</v-toolbar-title>
        <v-divider class="mx-4" inset vertical></v-divider>
        <v-spacer></v-spacer>
        <v-dialog v-model="dialog" max-width="500px">
          <template v-slot:activator="{ on, attrs }">
            <v-btn color="primary" dark class="mb-2" v-bind="attrs" v-on="on">
              New Transaction Detail
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
                      <v-select v-model="editedItem.transaction_id"
                        :items="transactionIDs"
                        filled
                        label="Transaction ID"
                        :rules="[rules.required]"
                      ></v-select>
                    </v-col>
                    <v-col cols="12">
                      <v-select v-model="editedItem.instrument_id"
                        :items="instrumentIDs"
                        filled
                        label="Instrument ID"
                        :rules="[rules.required]"
                      ></v-select>
                    </v-col>
                    <v-col cols="12">
                      <v-text-field
                        v-model="editedItem.quantity"
                        label="Quantity"
                        type="number"
                        :rules="[rules.greaterThanZero,rules.required]"
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
              <v-btn color="blue darken-1" text @click="deleteDetailBackend"
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
      { text: "Instrument ID", sortable: false, value: "instrument_id" },
      { text: "Quantity", sortable: false, value: "quantity" },
      { text: "Actions", value: "actions", sortable: false },
    ],
    details: [],
    editedIndex: -1,
    editedItem: {
      transaction_id: 0,
      instrument_id: 0,
      quantity: 0,
    },
    defaultItem: {
       transaction_id: 0,
       instrument_id: 0,
       quantity: 0,
    },
    transactionIDs : [],
    instrumentIDs : [],
  }),

  computed: {
    formTitle() {
      return this.editedIndex === -1 ? "New Transaction Detail " : "Edit Transaction Detail";
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
      this.getDetails();
      this.getTransactionIDs();
      this.getInstrumentIDs();
    },

    getTransactionIDs(){
      axios
        .get("http://localhost:8080/api/transaction_detail/TransactionIDs")
        .then((response) => {
          this.transactionIDs = response.data;
        });
    },

    getInstrumentIDs(){
      axios
        .get("http://localhost:8080/api/transaction_detail/InstrumentIDs")
        .then((response) => {
          this.instrumentIDs = response.data;
        });
    },

    getDetails() {
      axios
        .get("http://localhost:8080/api/transaction_detail")
        .then((response) => {
          this.details = response.data;
        });
    },

    deleteDetailBackend() {
      axios
        .delete(
          `http://localhost:8080/api/transaction_detail?transaction_id=${this.editedItem.transaction_id}&instrument_id=${this.editedItem.instrument_id}`
        )
        .then(() => {
          this.deleteItemConfirm();
        });
    },

    editItem(item) {
      this.editedIndex = this.details.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialog = true;
    },

    deleteItem(item) {
      this.editedIndex = this.details.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialogDelete = true;
    },

    deleteItemConfirm() {
      this.details.splice(this.editedIndex, 1);
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
          `http://localhost:8080/api/transaction_detail?&transaction_id=${this.editedItem.transaction_id}&instrument_id=${this.editedItem.instrument_id}&quantity=${this.editedItem.quantity}`
        )
        .then(() => {
          //this.categories.push(response.body);
          this.getDetails();
          this.close();
        });
    },

    updateBackend(){
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
    }
  },
};
</script>