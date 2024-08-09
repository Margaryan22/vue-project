<script setup>
import DrawerHead from "./DrawerHead.vue";
import CartItemList from "./CartItemList.vue";
import axios from "axios";
import { computed, ref, provide, watch } from "vue";
import InfoBlock from "./InfoBlock.vue";

const { cart, closeDrawer } = provide("cart");
const cartIsEmpty = computed(() => cart.value.length === 0);

const props = defineProps({
  totalPrice: Number,
  vatPrice: Number,
});

const buttonDisabled = computed(() => {
  return isCreating.value || cartIsEmpty.value;
});

const isCreating = ref(false);
const createOrder = async () => {
  try {
    isCreating.value = true;
    const { data } = await axios.post(
      "https://27ca0e96a42febf5.mokky.dev/orders",
      { items: cart.value, totalPrice: props.totalPrice.value }
    );
    cart.value = [];
    return data;
  } catch (error) {
    console.log(error);
  } finally {
    isCreating.value = false;
  }
};
</script>
<template>
  <div class="fixed top-0 left-0 h-full w-full bg-black z-10 opacity-70"></div>
  <div class="bg-white w-96 h-full fixed right-0 top-0 z-20 p-8">
    <DrawerHead />
    <div v-if="!totalPrice" class="flex h-full items-center">
      <InfoBlock
        title="Корзина пустая"
        description="Добавьте хотя бы одну пару кроссовок,чтобы сделать заказ"
        imgageUrl="/package.png"
      />
    </div>
    <div v-else>
      <CartItemList />

      <div v-if="totalPrice" class="flex flex-col gap-4 mt-7 bottom-0">
        <div class="flex gap-2">
          <span>Итого:</span>
          <div class="flex-1 border-b border-dashed"></div>
          <b>{{ totalPrice }} руб. </b>
        </div>

        <div class="flex gap-2">
          <spa n>Налог 5%:</spa>
          <div class="flex-1 border-b border-dashed"></div>
          <b>{{ vatPrice }}руб. </b>
        </div>
        <button
          :disabled="buttonDisabled"
          @click="createOrder"
          class="mt-4 bg-lime-500 w-full rounded-xl py-3 text-white disabled:bg-slate-300 cursor-pointer hover:bg-lime-600 active:bg-lime-700 transition font-bold"
        >
          Офомить заказ
        </button>
      </div>
    </div>
  </div>
</template>
