<template>
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/4.7.0/css/font-awesome.min.css" />
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
                <select name="select" defaultValue="Burguer" v-model="plate.name" @change="changeImage"
                  placeholder="Select a plate">
                  <option v-for="(name, index) in names" :value="name" :key="index">
                    {{ name }}
                  </option>
                </select>
              </div>
              <div class="form-group">
                <img :src="plate.image" :alt="plate.name" />
              </div>

              <div class="form-group">
                <input type="number" v-model.number="plate.price" placeholder="price of plate" name="price" required
                  value="" @input="validatePositiveNumber($event, 'price')" />
              </div>
              <div class="form-group">
                <input type="number" v-model.number="plate.stock" placeholder="Number of plates" name="stock" required
                  value="" @input="validateStock($event, 'stock')" />
              </div>

              <div class="form-group">
                <input required type="submit" value="Add plate" class="submit-button" />
              </div>
            </div>
          </form>
        </div>
      </div>
    </section>


    <section class="search" v-if="visibleSearch">
      <input type="text" placeholder="Search" @input="searchPlates" />
    </section>
    <section class="plate-list">
      <div v-for="plate in plates" :key="plate.id" class="plate-item">
        <div class="plate-item-image">
          <img :src="plate.image" :alt="plate.name" />
        </div>
        <div class="plate-item-details">
          <h2>{{ plate.name }}</h2>
          <div class="une">
            <p>Unit price: <i class="fa fa-usd"></i>{{ plate.price }}</p>
            <p>Stock: {{ plate.stock }}</p>
          </div>
          <p>Total value: <i class="fa fa-usd"></i>{{ plate.valorTotal }}</p>
          <div class="plate-item-buttons">
            <button @click="openDeleteModal(plate.id)"><i class="fa fa-trash"></i></button>
            <div v-if="showDeleteModal[plate.id]" class="modal-overlay">
              <div class="modal">
                <div class="modal-content">
                  <p>you want to remove all stock or just one product?</p>
                  <div class="modal-buttons">
                    <div class="two">
                      <button @click="deletePlate(plate.id)">Remove all Stock</button>
                      <button @click="deleteOnePlate(plate)">Remove a Stock</button>
                      <button @click="AddStock(plate)">Add a Stock</button>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </section>
    <div class="show-total-count" v-if="plates.length > 0">
      <button class="submit-button" @click="openCountModal()">
        Show total count
      </button>
      <div class="plate-item-buttons">
        <div v-if="showCountModal" class="modal-overlay">
          <div class="modal">
            <div class="modal-content">

              <p v-if="getTotalCount() > 0">Total count: {{ getTotalCount() }}</p>
              <p v-else>You don't have pays pending
              </p>
              <div class="modal-buttons">
                <button @click="closeCountModal()">Pay</button>
                <button @click="closeCountModalA()">After</button>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </main>
</template>

<script>
import { ref } from "vue";
import staticPlates from "../utils/plates.js";

export default {
  name: "PlatesForm",
  setup() {
    const showDeleteModal = ref([]);
    const showCountModal = ref(false);
    const platesNames = staticPlates.map((food) => food.name);
    const plates = ref([]);
    const visibleSearch = ref(false);
    /*plantilla e estructura de los platos*/
    const plate = ref({
      id: 0,
      name: platesNames[0],
      price: "",
      stock: "",
      image: "",
      valorTotal: 0,
    });
    /*copia para manejar los platos en busqueda y que no se eliminen los originales*/
    let copyPlates = [];

    /*pone la primer imagen en la plantilla*/
    plate.value.image = staticPlates[0].image;

    /*cambia la imagen dependiendo de la seleccionada*/
    const changeImage = () => {
      const selectedPlate = staticPlates.find(
        (food) => food.name === plate.value.name
      );
      if (selectedPlate) {
        plate.value.image = selectedPlate.image;
      } else {
        plate.value.image = "";
      }
    };

    /*reset el form para que el suario pueda crear otro plato*/
    const resetPlate = (id) => {
      plate.value = {
        id: id,
        name: platesNames[0],
        price: "",
        stock: "",
        image: staticPlates[0].image,
      };
    };

    /*funcion para eliminar un plato completo guardando y setiando platos y copyPLates*/
    const deletePlate = (id) => {
      updatePlates();
      cleanSearch();
      plates.value = plates.value.filter((plate) => plate.id !== id);
      updateCopyPlates();
      closeDeleteModal(id);
      if (plates.value.length === 0) {
        visibleSearch.value = false;
      }
    }
    /*funcion para agregar un plato a la lista de platos*/
    const AddStock = (plate) => {
      updatePlates();
      cleanSearch();
      plate.stock += 1;
      plate.valorTotal = plate.price * plate.stock;
      updateCopyPlates();
      closeDeleteModal(plate.id);
    };
    /*funcion para eliminar un plato de la lista de platos mas bien un stock se podria decir*/
      const deleteOnePlate = (plate) => {
        if (plate.stock > 0) {
          updatePlates();
          cleanSearch();
          plate.stock -= 1;
          plate.valorTotal = plate.price * plate.stock;
          updateCopyPlates();
          if (plate.stock === 0) {
            deletePlate(plate.id);
          }
        }
        closeDeleteModal(plate.id);
      };
      /*funcion para crear un plato y agregarlo a la lista de platos*/
        const createPlate = () => {
          plate.value.id++;
          const newPlate = { ...plate.value, valorTotal: plate.value.price * plate.value.stock };
          plates.value.push(newPlate);
          copyPlates.push(newPlate);
          resetPlate(plate.value.id);
          visibleSearch.value = true;
        };
      /*funcion para validar que el numero sea positivo*/
    const validatePositiveNumber = (event, field) => {
      if (event.target.value < 0) {
        event.target.value = 0;
        plate.value[field] = 0;
      }
    };
    /*fvalidacion para que el numero de platos sea mayor a 1 */
    const validateStock = (event) => {
      const value = parseInt(event.target.value);
      if (isNaN(value) || value < 1) {
        event.target.value = "";
    };
  };

  /*funcion para limpiar el campo de busqueda*/
    const cleanSearch = () => {
      document.querySelector(".search input").value = "";
    };

    /*funcion para eliminar todos los platos*/
    const deleteAllPlates = () => {
      plates.value = [];
      copyPlates = [];
      visibleSearch.value = false;
    }
    /*funcion para actualizar la copia de los platos*/
    const updateCopyPlates = () => {
      copyPlates = [...plates.value];
    };
    /*funcion para actualizar los platos*/
    const updatePlates = () => {
      plates.value = [...copyPlates];
    };
    /*funcion para buscar los platos*/
    const searchPlates = () => {
      const search = document.querySelector('.search input').value.toLowerCase();
      if (search.length > 0) {
        const filteredPlates = copyPlates.filter((plate) =>
          plate.name.toLowerCase().includes(search)
        );
        plates.value = filteredPlates;
      } else {
        plates.value = [...copyPlates];
      }
    };

    /*funcion para obtener el total de la cuenta*/
    const getTotalCount = () => {
      return copyPlates.reduce((total, plate) => total + plate.valorTotal, 0);
    };

    /*funcion para abrir el modal de cuenta*/
    const openCountModal = () => {
      showCountModal.value = true;
    };
    /*funcion para cerrar el modal de cuenta*/
    const closeCountModal = () => {
      showCountModal.value = false;
      if (getTotalCount() != 0) {
        deleteAllPlates();
      }
    };

    /*funcion para cerrar el modal de cuenta cuando desea pagar despues*/
    const closeCountModalA = () => {
      showCountModal.value = false;
    };
    /*funcion para abrir el modal de eliminar plato*/
    const openDeleteModal = (id) => {
      showDeleteModal.value[id] = true;
    };
    /*funcion para cerrar el modal de eliminar plato*/
    const closeDeleteModal = (id) => {
      showDeleteModal.value[id] = false;
    };

    return {
      plate,
      plates,
      createPlate,
      sPlates: staticPlates,
      names: platesNames,
      changeImage,
      deletePlate,
      validatePositiveNumber,
      showDeleteModal,
      deleteOnePlate,
      closeDeleteModal,
      openDeleteModal,
      openCountModal,
      closeCountModal,
      showCountModal,
      visibleSearch,
      searchPlates,
      AddStock,
      getTotalCount,
      updateCopyPlates,
      validateStock,
      closeCountModalA,

    };
  },
};

</script>

<style>
:root {
  --main_color: #ff8000;
  --second_color: rgb(234, 233, 233);
}

main::before {
  content: "";
  background-image: url(../public/mainfond.jpg);
  background-size: cover;
  /* Ajusta el tamaño de la imagen para cubrir todo el fondo */
  background-attachment: fixed;
  /* Fija la imagen en su posición mientras se desplaza la página */
  opacity: 0.03;
  /* Ajusta la opacidad según tus preferencias */
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: -1;
  /* Coloca la imagen detrás del contenido */
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

.submit-button:hover {
  transform: scale(1.1);
  transition: all 3s ease;
}

.show-total-count button {
  margin: 0 auto;
  display: block;
  text-align: center;
}

.submit-button {
  width: 30%;
  font-size: 20px;
  font-weight: 700;
  outline: none;
  border: none;
  cursor: pointer;
  text-align: center;
  background-color: #ff8000;
  color: #ffffff;
  box-shadow: none;
  border-radius: 25px;
  padding: 20px;
}

.form-group input,
select ::placeholder {
  color: white;
}

.form-group select {
  color: white;
  color: white
}

.form-group select {
  color: white
}

/* Chrome, Safari, Edge, Opera */
input::-webkit-outer-spin-button,
input::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

/* Firefox */
input[type="number"] {
  -moz-appearance: textfield;
}

select {
  color: white;
  background-color: black;
  border: none;
  padding: 20px 10px 20px 5px;
  font-size: 18px;
  width: 100%;
}

/* Estilo para todas las opciones */
option {
  background-color: black;
  color: white;
}

/*lista de platos*/
.plate-list {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 20px;
  margin-top: 20px;
  margin-bottom: 4rem;
}

.plate-item {
  display: grid;
  grid-template-rows: auto auto auto;
  /* Tres filas de altura automática */
  justify-content: center;
  /* Centra los elementos horizontalmente */
  align-items: center;
  /* Centra los elementos verticalmente */
  overflow: hidden;
  background-color: var(--second_color);
  color: black;
  border-radius: 2.5rem;
  box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.2);
}

.plate-item-image img {
  width: 100%;
  height: auto;
  border-top-left-radius: 10px;
  border-top-right-radius: 10px;
}

.plate-item-details {
  display: flex;
  flex-direction: column;
  text-align: center;
  font-size: 1.2rem;
}

.plate-item-details h2 {
  grid-row: 1;
  text-align: center;
}


.une {
  display: flex;
  justify-content: space-around;
  align-items: center;
  margin: 10px 0;
}

.une p {
  font-size: 1.2rem;
}

.plate-item-buttons {
  grid-row: 3;
  display: flex;
  justify-content: center;
  align-items: center;
  margin-bottom: 10px;
}

.plate-item-buttons button {
  background-color: var(--main_color);
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 0.4rem;
  cursor: pointer;
  transition: all 0.3s ease;
  font-size: 1.6rem;
  margin-top: 0.4rem;
}

.plate-item-buttons button:hover {
  transform: scale(1.1);
  transition: all 0.3s ease;
}

/*lista de platos*/

/*stylos modal*/
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 2;
}

.modal {
  background-color: white;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.5);
  z-index: 2;
  color: black;
  font-size: 1.2rem;
}
.modal-buttons {
  display: flex;
}

.modal-buttons button {
  padding: 10px 20px;
  font-size: 1.3rem;
  margin-top: 0.4rem;
  border: none;
  margin-right: 10px; 
}

.modal-buttons button:last-child {
  margin-right: 0;
}


.two {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin: 1rem;

}

.two button {
  margin: 0 10px;
}


.modal-content p {
  font-size: 1.2rem;
  text-align: center;
  margin-top: 1rem;
  font-weight: 700;
}

/*stylos modal*/

/*stylos search*/
.search {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-top: 20px;
}

.search input {
  width: 35%;
  margin: auto;
  padding: 10px;
  margin: 10px 0;
  border-radius: 10px;
  border: none;
  outline: none;
  font-size: 1.2rem;
  background-color: var(--second_color);
  color: black;
}

/*stylos search*/
</style>
