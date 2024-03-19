<template>
  <main>
    <section class="section-wrapper">
      <div class="box-wrapper">
        <div class="info-wrap">
          <h2 class="info-title">Calculate your order</h2>
        </div>
        <div class="form-wrap">
          <form @submit.prevent="createPlate">
            <div class="form-fields">
              <div class="form-group">
                <select
                  name="select"
               defaultValue="Burguer"
                  v-model="plate.name"
                  @change="changeImage"
                >
                  <option
                    v-for="(name, index) in names"
                    :value="name"
                    :key="index"
                  >
                    {{ name }}
                  </option>
                </select>
              </div>
              <div class="form-group">
                <img :src="image" alt="" />
              </div>

              <div class="form-group">
                <input
                  type="number"
                  v-model.number="plate.price"
                  placeholder="price of plate"
                  name="price"
                  required
                  value="null"
                />
              </div>
              <div class="form-group">
                <input
                  type="number"
                  v-model.number="plate.stock"
                  placeholder="Number of plates"
                  name="stock"
                  required
                  value="null"
                />
              </div>

              <div class="form-group">
                <input
                  required
                  type="submit"
                  value="Add plate"
                  class="submit-button"
                />
              </div>
            </div>
          </form>
        </div>
      </div>
    </section>

    <h1>Plate list</h1>
    <div v-for="item in plates" :key="item.id">
      <h2>{{ item.name }}</h2>
      <p>Price: {{ item.price }}</p>
      <p>Stock: {{ item.stock }}</p>
    </div>

    <h1>Plate list</h1>
    <div v-for="item in sPlates">
      <h2>{{ item }}</h2>
    </div>
  </main>
</template>

<script>
import { ref } from "vue";
import staticPlates from "../utils/plates.js";

export default {
  name: "PlatesForm",
  setup() {
    const platesNames = staticPlates.map((food) => food.name);
    const image = ref(staticPlates[0].image);
    const images = staticPlates.map((food) => food.image);

    const plates = ref([]);
    const plate = ref({
      id: 0,
      name: "",
      price: 0,
      stock: 0,
    });

    const changeImage = () => {
      const index = staticPlates.findIndex(
        (food) => food.name === plate.value.name
      );
      image.value = staticPlates[index].image;
    };

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
      createPlate,
      sPlates: staticPlates,
      names: platesNames,
      image,
      images,
      changeImage,
    };
  },
};
</script>

<style scoped>
main::before {
  content: "";
  background-image: url(../public/mainfond.jpg);
  background-size: cover; /* Ajusta el tamaño de la imagen para cubrir todo el fondo */
  background-attachment: fixed; /* Fija la imagen en su posición mientras se desplaza la página */
  opacity: 0.03; /* Ajusta la opacidad según tus preferencias */
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: -1; /* Coloca la imagen detrás del contenido */
}

.section-wrapper {
  width: 100%;
  height: 100vh;
  padding: 10px;

  display: flex;
  justify-content: center;
  align-items: center;
}
.box-wrapper {
  position: relative;
  display: table;
  width: 1200px;
  margin: auto;
  margin-top: 35px;
  border-radius: 30px;
  justify-content: center;
  align-items: center;
}

.info-wrap {
  display: flex;
  justify-content: center;
  align-items: center;
}

.form-wrap {
  width: 100%;

  padding: 40px 25px 35px 25px;
  border-radius: 0px 30px 30px 0px;
  background: transparent;
}

.form-fields {
  display: flex;
  flex-direction: column;
  width: 50%;
  padding: 15px 5px 5px 5px;
}
.form-fields input,
.form-fields select {
  border: none;
  outline: none;
  background: none;
  font-size: 18px;
  color: #000000;
  padding: 20px 10px 20px 5px;
  width: 100%;
}

.form-group {
  float: left;
  padding: 0px 30px;
  margin: 14px 12px;
  border-radius: 25px;
  box-shadow: inset 8px 8px 8px #cbced1, inset -8px -8px 8px #ffffff;
}

.form-fields .form-group:nth-child(2) {
  border: none;
  outline: none;
  background: none;
  box-shadow: none;
  position: absolute;
  top: 105px;
  right: 100px;
  width: 30rem;
}

.form-group:nth-child(2) img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  border-radius: 25px;
}

.form-group:nth-child(5) {
  background-color: #ff8000;
  color: #ffffff;
  box-shadow: none;
}

.form-group:nth-child(5):hover {
  transform: scale(1.1);
  transition: all 3s ease;
}

.submit-button {
  width: 30%;
  font-size: 20px;
  font-weight: 700;
  outline: none;
  border: none;
  cursor: pointer;

  text-align: center;
  background-color: #1a3492;
}
</style>
