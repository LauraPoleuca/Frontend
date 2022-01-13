<template>
  <v-data-table :headers="headers" :items="instruments" class="elevation-1">
    <template v-slot:top>
      <v-toolbar flat>
        <v-toolbar-title>Instrument</v-toolbar-title>
        <v-divider class="mx-4" inset vertical></v-divider>
        <v-spacer></v-spacer>
        <v-dialog v-model="dialog" max-width="500px">
          <template v-slot:activator="{ on, attrs }">
            <v-btn color="primary" dark class="mb-2" v-bind="attrs" v-on="on">
              New Instrument
            </v-btn>
          </template>
          <v-card>
            <v-card-title>
              <span class="text-h5">{{ formTitle }}</span>
            </v-card-title>

            <v-card-text>
              <v-container>
                <v-row>
                  <v-col cols="12">
                    <v-select v-model="editedItem.category_id"
                      :items="CategoryIDs"
                      filled
                      label="Category ID"
                    ></v-select>
                  </v-col>
                  <v-col cols="12">
                    <v-text-field
                      v-model="editedItem.colour"
                      label="Colour"
                      :rules="[rules.required, rules.counter]"
                    ></v-text-field>
                  </v-col>
                  <v-col cols="12">
                    <v-text-field
                      v-model="editedItem.price"
                      label="Price"
                    ></v-text-field>
                  </v-col>
                  <v-col cols="12">
                    <v-text-field
                      v-model="editedItem.instrument_name"
                      label="Name"
                      :rules="[rules.required, rules.counter]"
                    ></v-text-field>
                  </v-col>
                </v-row>
              </v-container>
            </v-card-text>

            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" text @click="close"> Cancel </v-btn>
              <v-btn color="blue darken-1" text @click="save"> Save </v-btn>
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
              <v-btn color="blue darken-1" text @click="deleteInstrumentBackend"
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
    },
    dialog: false,
    dialogDelete: false,
    headers: [
      {
        text: "Instrument ID",
        align: "start",
        sortable: false,
        value: "instrument_id",
      },
      { text: "Category ID", sortable: false, value: "category_id" },
      { text: "Colour", sortable: false, value: "colour" },
      { text: "Price", sortable: false, value: "price" },
      { text: "Name", sortable: false, value: "instrument_name" },
      { text: "Actions", value: "actions", sortable: false },
    ],
    instruments: [],
    CategoryIDs:[],
    editedIndex: -1,
    editedItem: {
      instrument_id: 0,
      category_id: 0,
      colour: "",
      price: 0,
      instrument_name: "",
    },
    defaultItem: {
      instrument_id: 0,
      category_id: 0,
      colour: "",
      price: 0,
      instrument_name: "",
    },
  }),

  computed: {
    formTitle() {
      return this.editedIndex === -1 ? "New Instrument" : "Edit Instrument";
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
    getCategoryIDs(){
      axios.get("http://localhost:8080/api/instrument/CategoryIDs").then((response) => {
        this.CategoryIDs = response.data;
      });

    },
    initialize() {
      this.getInstruments();
      this.getCategoryIDs();
    },

    getInstruments() {
      axios.get("http://localhost:8080/api/instrument").then((response) => {
        this.instruments = response.data;
      });
    },

    deleteInstrumentBackend() {
      axios
        .delete(
          `http://localhost:8080/api/instrument?instrument_id=${this.editedItem.instrument_id}`
        )
        .then(() => {
          this.deleteItemConfirm();
        });
    },

    editItem(item) {
      this.editedIndex = this.instruments.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialog = true;
    },

    deleteItem(item) {
      this.editedIndex = this.instruments.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialogDelete = true;
    },

    deleteItemConfirm() {
      this.instruments.splice(this.editedIndex, 1);
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
      } else {
        //CREATE
        this.createBackend();
      }
    },

    createBackend() {
      axios
        .post(
          `http://localhost:8080/api/instrument?&category_id=${this.editedItem.category_id}&colour=${this.editedItem.colour}&price=${this.editedItem.price}&instrument_name=${this.editedItem.instrument_name}`
        )
        .then(() => {
          //this.categories.push(response.body);
          this.getInstruments();
          this.close();
        });
    },

    /*updateBackend() {
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
    },*/
  },
};
</script>