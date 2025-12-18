<script setup>
import { useQuasar } from "quasar";
import { ref } from "vue";

import productosData from "../components/mercancia.json";

const $q = useQuasar();

const contador = ref(0);

const productos = ref(productosData);
</script>

<template>
  <div id="dad">
    <div id="container">
      <div id="productos">
        <h1 class="titulo">Productos</h1>
        <div v-for="item in productos" :key="item.id" class="productos">
          <div class="bts">
            <div>
              <p class="text">
                {{ item.nombre }}
              </p>
            </div>
            <div>
              <p class="text" style="color: darkgreen">${{ item.precio }}</p>
            </div>
          </div>
          <div class="">
            <button @click="$q.notify(`Agregado ${item.nombre}`)">
              Añadir
            </button>
          </div>
        </div>
      </div>

      <div id="carrito">
        <h1 class="titulo2">Carrito de Compras</h1>
        <span class="text2 span">🛻 Resumen del carrito {{ contador }} ✅ </span>
        <span class="text2 span">Total de productos: {{ contador }} items </span>
        <span class="text2 span">📋Subtotal: 💲{{ contador }} pesos </span>
        <span class="text2 span"> ℹ️impuesto (16%): 💲{{ contador }} pesos</span>
        <span class="titulo2 span"> Total a pagar: 💲{{ contador }} pesos </span>
      </div>
    </div>
  </div>
</template>

<style scoped>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  overflow: hidden;
}

#dad {
  background-color: #f5f5f5;
  display: flex;
  justify-content: center;
  align-items: center;
}

#container {
  width: 60vw;
  min-width: 350px;
  min-height: 100vh;
  display: grid;
  justify-content: center;
  align-items: center;
  background-color: white;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  border-radius: 8px;
  grid-template-columns: 1fr;
}

#productos {
  display: grid;
  grid-template-columns: 1fr;
}

.productos {
  display: grid;
  background-color: #ffd600;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  border-radius: 8px;
  grid-template-columns: 1fr;
  box-sizing: border-box;
  overflow: hidden;
  margin: 15px;
  padding: 10px 5px;
  text-align: center;
}

#carrito {
  display: grid;
  background-color: #e30613;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  border-radius: 8px;
  grid-template-columns: 1fr;
  margin: 15px;
  box-sizing: border-box;
  overflow: hidden;
  padding: 10px 5px;
  text-align: center;
}

.bts {
  display: grid;
  grid-template-columns: 2fr 1fr;
  align-items: center;
  justify-items: center;
}

.text {
  font-family: Arial, sans-serif;
  font-size: 1.2em;
  color: #333;
  font-weight: bold;
}

.titulo {
  text-align: center;
  margin: 20px 0;
  font-family: Tahoma, Geneva, Verdana, sans-serif;
  color: #e30613;
    font-size: 5em;
}

.text2 {
  font-family: Arial, sans-serif;
  font-size: 1.2em;
  color: #ffffff;
  font-weight: bold;
}

.titulo2 {
  text-align: center;
  margin: 2px 0;
  font-family: Tahoma, Geneva, Verdana, sans-serif;
  color: #ffffff;
  font-size: 2em;
}
.span {
  background-color: #ffd600;
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  width: fit-content;
  padding: 5px 10px;
    margin: 4px auto;
    border: #f5f5f5 solid 1px;
}
</style>