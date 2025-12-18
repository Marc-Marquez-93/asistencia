<script setup>
import { ref, computed, watch } from "vue";
import { useQuasar } from "quasar";
import productosData from "../components/mercancia.json";

const $q = useQuasar();

const contador = ref(0);
const subtotal = ref(0);
const productos = ref(productosData);

const ultimoProducto = ref(null);

function estadoProducto(id) {
  const carrito = JSON.parse(sessionStorage.getItem('carrito')) || [];
  const prod = carrito.find(p => p.id === id);

  return {
    enCarrito: !!prod,        
    cantidad: prod ? prod.cantidad : 0 
  };
}

if (sessionStorage.getItem('contador') && sessionStorage.getItem('subtotal')) {
  contador.value = Number(sessionStorage.getItem('contador'));
  subtotal.value = Number(sessionStorage.getItem('subtotal'));
} else {
  sessionStorage.setItem('carrito', JSON.stringify([]));
}

function agregar(prod) {
  contador.value++;
  const precio = Number(prod.precio.replace(/\./g, ""));
  subtotal.value += precio;

  ultimoProducto.value = { ...prod, accion: 'agregar' };
}

function quitar(prod) {
  contador.value--;
  const precio = Number(prod.precio.replace(/\./g, ""));
  subtotal.value -= precio;

  ultimoProducto.value = { ...prod, accion: 'quitar' };
}

watch([contador, subtotal, ultimoProducto], ([con, sub, prod]) => {
  sessionStorage.setItem('contador', con);
  sessionStorage.setItem('subtotal', sub);

  if (prod) {
    let carrito = JSON.parse(sessionStorage.getItem('carrito')) || [];

    if (prod.accion === 'agregar') {
      const index = carrito.findIndex(item => item.id === prod.id);
      if (index !== -1) {
        carrito[index].cantidad += 1;
      } else {
        carrito.push({
          id: prod.id,
          nombre: prod.nombre,
          precio: prod.precio,
          cantidad: 1
        });
      }

      $q.notify({
        message: `${prod.nombre} agregado al carrito!!! 😍`,
        color: 'positive',
        position: 'top',
        icon: 'check',
        timeout: 2000
      });

    } else if (prod.accion === 'quitar') {
      const index = carrito.findIndex(item => item.id === prod.id);
      if (index !== -1) {
        carrito[index].cantidad -= 1;
        if (carrito[index].cantidad <= 0) {
          carrito.splice(index, 1); 
        }
      }

      $q.notify({
        message: `${prod.nombre} eliminado del carrito 😢`,
        color: 'negative',
        position: 'top',
        icon: 'close',
        timeout: 2000
      });
    }

    sessionStorage.setItem('carrito', JSON.stringify(carrito));
    ultimoProducto.value = null;
  }
});

const subtotalFinal = computed(() => subtotal.value);
const contadorFinal = computed(() => contador.value);
const impuesto = computed(() => subtotalFinal.value * 0.16);
const total = computed(() => subtotalFinal.value + impuesto.value);

watch(total, (tot) => {
  if(tot>999999) {
    $q.dialog({
    title: 'Envio gratiiiiis 🥹💵💵',
    message: 'Te tenemos grandes noticias, superaste $1.000.000 de pesos, por lo que te mereces un envio gratis 😁',
    ok: {
      label: 'Aceptar',
      color: 'positive'
    },
  })
  }
})

function formatoMoneda(num) {
  return num.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ".");
}
</script>


<template>
  <div id="dad">
    <div id="container">
      <div id="productos">
        <h1 class="titulo no-select">Tienda Totto</h1>
        <div v-for="item in productos" :key="item.id" class="productos">
          <div class="bts">
            <div>
              <p class="text no-select">
                {{ item.nombre }}
              </p>
            </div>
            <div>
              <p class="text no-select" style="color: darkgreen">${{ item.precio }}</p>
            </div>
          </div>
          <div v-if="estadoProducto(item.id).cantidad === 0">
            <q-btn square color="positive" glossy icon="local_grocery_store" label="agregar al carrito"
              @click="agregar(item)" />
          </div>
          <div v-else>
            <q-btn round color="positive" glossy icon="add" style="margin: 5px;" @click="agregar(item)" />
            <span>{{ estadoProducto(item.id).cantidad }}</span>
            <q-btn round color="deep-orange" glossy icon="remove" style="margin: 5px;" @click="quitar(item)" />
          </div>
        </div>
      </div>

      <div id="carrito">
        <h1 class="titulo2 no-select">Carrito de Compras</h1>
        <span class="text2 span no-select">🛻 Resumen del carrito {{ contadorFinal }} ✅ </span>
        <span class="text2 span no-select">Total de productos: {{ contadorFinal }} items </span>
        <span class="text2 span no-select">📋Subtotal: 💲{{ formatoMoneda(subtotalFinal) }} pesos </span>
        <span class="text2 span no-select"> ℹ️impuesto (16%): 💲{{ formatoMoneda(impuesto) }} pesos</span>
        <span class="titulo2 span no-select"> Total a pagar: 💲{{ formatoMoneda(total) }} pesos </span>
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
  background: linear-gradient(to bottom, #ffd600, #e30613);
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

.no-select {
  user-select: none;
}
</style>