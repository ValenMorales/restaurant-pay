<template>
  <form @submit.prevent="createPlate">
    <input type="text" v-model="plate.name" placeholder="Plate name" name="name" required/>
    <input type="number" v-model.number="plate.price" placeholder="Plate price" name="price" required/>
    <input type="number" v-model.number="plate.stock" placeholder="Number of plates" name="stock" required/>
    
    <button type="submit">Add plate</button>
  </form>

  <h1>Plate list</h1>
  <div v-for="item in plates" :key="item.id">
    <h2>{{ item.name }}</h2>
    <p>Price: {{ item.price }}</p>
    <p>Stock: {{ item.stock }}</p>
  </div>
</template>

<script>
import { ref } from "vue";

export default {
  name: "PlatesForm", 
  setup() {
    const plates = ref([]);
    const plate = ref({
      id: 0,
      name: "",
      price: 0,
      stock: 0,
    });

    const createPlate = () => {
      plate.value.id++;
      plates.value.push({ ...plate.value });
      plate.value = {
        id: plate.value.id,
        name: "",
        price: 0,
        stock: 0,
      };
    };

    return {
      plate,
      plates,
      createPlate
    };
  },
};
</script>

<style scoped>
h1 {
  background-color: aqua;
}
</style>./PlatesForm.vue
