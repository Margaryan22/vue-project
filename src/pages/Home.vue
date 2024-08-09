<script setup>
import { reactive, watch, ref, onMounted, inject } from "vue";
import axios from "axios";
import CardList from "../components/CardList.vue";
const { addToCart, cart, removeFromCart } = inject("cart");
const items = ref([]);
const filters = reactive({
  //объект из фильтров для добавления в строку url при поиске конкретных товаров
  //для объектов лучше reactive
  sortyBy: "title",
  searchQuery: "",
});
//функция следит за изменением select
const onChangeSelect = (event) => {
  filters.sortyBy = event.target.value;
};

//функция следит за изменением input
const onChangeSearchInput = (event) => {
  filters.searchQuery = event.target.value;
};

const onClickAddPlus = (item) => {
  if (!item.isAdded) {
    addToCart(item);
  } else {
    removeFromCart();
  }
};
const fetchFavorites = async () => {
  const { data: favorites } = await axios.get(
    `https://27ca0e96a42febf5.mokky.dev/favorites`
  );

  items.value = items.value.map((item) => {
    const favorite = favorites.find((favorite) => favorite.itemId === item.id);

    if (!favorite) {
      return item;
    }

    return {
      ...item,
      isFavorite: true,
      favoriteId: favorite.id,
    };
  });
};

const addToFavorite = async (item) => {
  try {
    item.isFavorite = !item.isFavorite;

    if (!item.isFavorite) {
      const obj = {
        itemId: item.id,
      };
      const { data } = await axios.post(
        `https://27ca0e96a42febf5.mokky.dev/favorites`,
        obj
      );

      item.favoriteId = data.id;
    } else {
      await axios.delete(
        `https://27ca0e96a42febf5.mokky.dev/favorites/${item.favoriteId}`
      );

      item.favoriteId = null;
    }
  } catch (err) {
    console.log(err);
  }
};

watch(
  cart,
  () =>
    (items.value = items.value.map((item) => ({
      ...item,
      isAdded: false,
    })))
);

onMounted(async () => {
  const localCart = localStorage.getItem("cart");
  cart.value = localCart ? JSON.parse(localCart) : [];

  await fetchItems();
  await fetchFavorites();
  items.value = items.value.map((item) => ({
    ...item,
    isAdded: cart.value.some((cartItem) => cartItem.id === item.id),
  }));
}); //первый запрос при открытии сайта

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
watch(filters, fetchItems); //смотрит за состоянием фильтра,когда меняется значение
</script>

<template>
  <div class="flex justify-between items-center">
    <h2 class="text-3xl font-bold mb-8">Все кроссовки</h2>
    <div class="flex gap-4">
      <select
        @change="onChangeSelect"
        class="py-2 px-3 border rounded-md outline-none"
      >
        <option value="name">По названию</option>
        <option value="price">По цене(дешевые)</option>
        <option value="-price">По цене(дорогие)</option>
      </select>
      <div class="relative">
        <img class="absolute left-4 top-3" src="/search.svg" />
        <input
          @input="onChangeSearchInput"
          class="border rounded-md py-2 pl-11 pr-4 outline-none focus:border-gray-400"
          placeholder="Поиск..."
        />
      </div>
    </div>
  </div>

  <div class="mt-10">
    <CardList
      :items="items"
      @add-to-favorite="addToFavorite"
      @add-to-cart="onClickAddPlus"
    />
  </div>
</template>
