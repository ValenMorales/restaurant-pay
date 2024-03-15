<script>
import {ref, computed} from 'vue'
export default {
setup(){

const dado1= ref(0);
const dado2= ref(0);

const resultado = computed(()=> (dado1.value == dado2.value)? "Ganaste":"Perdiste");

const tirarDados =() =>{
dado1.value= Math.floor(Math.random() * 6+1);
dado2.value= Math.floor(Math.random() * 6+1);
}
return {dado1, dado2, resultado, tirarDados}

}

}
</script>

<template>
  <h1>Juego de dados</h1>

  <button @click="tirarDados">Tirar dados</button>
  <h2>Dado 1: {{ dado1 }}</h2>
  <h2>Dado 2: {{ dado2 }}</h2>

  <h1>{{ resultado }}</h1>
</template>

<style scoped>

</style>
