<template>
  <div>
    <v-treeview dense :items="items">
      <template v-slot:label="{ item }">
        <span>{{ item.name }}{{ " |  Цена:" }}{{ item.price }}</span>
      </template>
      <template v-slot:append="{ item }">
        <v-btn
          v-if="item.children.length == 0"
          color="pink"
          fab
          dark
          small
          right
          @click="handleInsert(item)"
        >
          +
        </v-btn>
      </template>
    </v-treeview>
    <v-card class="mx-auto" max-width="400" tile>
      <v-list-item v-for="item in basket" :key="item.id">
        <v-list-item-content>
          <v-list-item-title>{{item.name}}{{" | "}}{{item.price}}</v-list-item-title>
        </v-list-item-content>
      </v-list-item>
    </v-card>
  </div>
</template>

<script>
import axios from "axios";
export default {
  data: () => ({
    items: [],
    listOfParentCategories: [],
    showCreateDialog: false,
    newItem: {
      id: 0,
      name: "",
      price: 0,
      parentId: 0,
      children: null,
    },
    alert: false,
    basket:[],
  }),
  methods: {
    resetNewItem() {
      this.newItem = {
        id: 0,
        name: "",
        price: 0,
        parentId: 0,
        children: null,
      };
    },
    async getCatalogList() {
      await axios.get(`catalog`).then((resp) => {
        this.items = this.listToTree(resp.data);
      });
    },
    async getCatalogList_2() {
      await axios.get(`catalog`).then((resp) => {
        this.listOfParentCategories = resp.data;
      });
    },
    async createNewItem() {
      await axios.post(`catalog`, this.newItem).then((resp) => {
        this.alert = true;
        setTimeout(() => {
          this.alert = false;
        }, 2000);
        this.resetNewItem();
      });
    },
    handleInsert(item) {this.basket.push(item)},
    listToTree(list) {
      let map = {},
      roots = []
      list.forEach((element, index) => {
        element.children = []
        map[element.id] = index;
        if (element.parentId !== 0) 
        list[map[element.parentId]].children.push(element);
        else roots.push(element);
      });
      return roots;
    },
  },
  async created() {
    await this.getCatalogList();
  },
  watch: {
    showCreateDialog() {
      if (this.showCreateDialog) this.getCatalogList_2();
      else this.getCatalogList();
    },
  },
};
</script>