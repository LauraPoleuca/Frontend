<template>
  <v-data-table :headers="headers" :items="categories" class="elevation-1">
    <template v-slot:top>
      <v-toolbar flat>
        <v-toolbar-title>Instrument Category</v-toolbar-title>
        <v-divider class="mx-4" inset vertical></v-divider>
        <v-spacer></v-spacer>
        <v-dialog v-model="dialog" max-width="500px">
          <template v-slot:activator="{ on, attrs }">
            <v-btn color="primary" dark class="mb-2" v-bind="attrs" v-on="on">
              New Category
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
                    <v-text-field
                      v-model="editedItem.category_name"
                      label="Category Name"
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
              <v-btn color="blue darken-1" text @click="deleteCategoryBackend"
                >OK</v-btn
              >
              <v-spacer></v-spacer>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-toolbar>
    </template>
    <template v-slot:item.actions="{ item }">
      <v-icon small class="mr-2" @click="editItem(item)"> mdi-pencil </v-icon>
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
    dialog: false,
    dialogDelete: false,
    headers: [
      {
        text: "Category ID",
        align: "start",
        sortable: false,
        value: "category_id",
      },
      { text: "Name", sortable: false, value: "category_name" },
      { text: "Actions", value: "actions", sortable: false },
    ],
    categories: [],
    editedIndex: -1,
    editedItem: {
      category_id: 0,
      category_name: "",
    },
    defaultItem: {
      category_id: 0,
      category_name: "",
    },
  }),

  computed: {
    formTitle() {
      return this.editedIndex === -1 ? "New Category" : "Edit Category";
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
      this.getCategories();
    },

    getCategories() {
      axios
        .get("http://localhost:8080/api/instrument_category")
        .then((response) => {
          this.categories = response.data;
        });
    },

    deleteCategoryBackend() {
      axios
        .delete(
          `http://localhost:8080/api/instrument_category?category_id=${this.editedItem.category_id}`
        )
        .then(() => {
          this.deleteItemConfirm();
        });
    },

    editItem(item) {
      this.editedIndex = this.categories.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialog = true;
    },

    deleteItem(item) {
      this.editedIndex = this.categories.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialogDelete = true;
    },

    deleteItemConfirm() {
      this.categories.splice(this.editedIndex, 1);
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
          `http://localhost:8080/api/instrument_category?&category_name=${this.editedItem.category_name}`
        )
        .then(() => {
          //this.categories.push(response.body);
          this.getCategories();
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