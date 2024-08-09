<script setup>
import { onMounted } from "vue";
import axios from "axios";
import CardList from "../components/CardList.vue";

const favorites = ref([]);

onMounted(async () => {
  try {
    const { data } = await axios.get(
      "https://27ca0e96a42febf5.mokky.dev/favorites?_relations=items"
    );
    favorites.value = data.map((obj) => obj.item);
  } catch (error) {
    console.log(error);
  }
});
</script>
<template>
  <h1>Мои закладки</h1>
  <h2 class="text-3xl font-bold mb-8">Мои кроссовки</h2>
  <CardList :items="favorites" is-favorites />
</template>
