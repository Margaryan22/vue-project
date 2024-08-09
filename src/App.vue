<script setup>
defineProps({ totalPrice: Number });
import axios from "axios";
import Header from "./components/Header.vue";
import CardList from "./components/CardList.vue";
import Drawer from "./components/Drawer.vue";
import Home from "./pages/Home.vue";
import { ref, watch, provide, computed } from "vue";

//для массивов ref, массив данных для передачи информации в каждую карточку товаров
/*Корзина START*/
const cart = ref([]);
const drawerOpen = ref(false);
const totalPrice = computed(() =>
  cart.value.reduce((acc, item) => acc + item.price, 0)
);
const vatPrice = computed(() => Math.round((totalPrice.value * 5) / 100));

const openDrawer = () => {
  drawerOpen.value = true;
};

const closeDrawer = () => {
  drawerOpen.value = false;
};

const addToCart = (item) => {
  cart.value.push(item);
  item.isAdded = true;
};

const removeFromCart = (item) => {
  cart.value.splice(cart.value.indexOf(item), 1);
  item.isAdded = false;
};

watch(
  cart,
  () => {
    localStorage.setItem("cart", JSON.stringify(cart.value));
  },
  { deep: true }
);
provide("cart", { cart, closeDrawer, openDrawer, addToCart, removeFromCart });
/*Корзина  END*/

//функция для запроса на бэкенд
const fetchItems = async () => {
  try {
    const params = {
      sortBy: filters.sortyBy,
    };
    //проверяем строку
    if (filters.searchQuery) {
      params.title = `*${filters.searchQuery}*`;
    }
    const { data } = await axios.get(
      `https://27ca0e96a42febf5.mokky.dev/items`,
      { params }
    );
    items.value = data.map((obj) => ({
      ...obj, //взяли все свойства с объекта и в следующей строке прописываем дополнительные
      isFavorite: false,
      favoriteId: null,
      isAdded: false,
    }));
  } catch (error) {
    console.log(error);
  }
};
</script>

<template>
  <Drawer
    v-if="drawerOpen"
    :total-price="totalPrice"
    :vat-price="vatPrice"
    @create-order="createOrder"
  />
  <div class="bg-white w-4/5 m-auto rounded-xl shadow-xl mt-14">
    <Header :total-price="totalPrice" @open-drawer="openDrawer" />

    <div class="p-10">
      <router-view><Home /></router-view>
    </div>
  </div>
</template>
