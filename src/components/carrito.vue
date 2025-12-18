<script setup>
import { useQuasar  } from 'quasar'
import { ref, onMounted } from 'vue'

const $q = useQuasar()
const contador = ref(0)

function mostrarDialogo() {
  $q.dialog({
    title: '🍓 Mi Fruta Favorita',
    message: '¡Producto agregado al carrito!',
    ok: {
      label: 'Aceptar',
      color: 'primary'
    }
  })
}
function mostrarNotificacion() {
  $q.notify({
    message: '¡Producto agregado al carrito!',
    color: 'positive',     // 🔹 verde
    position: 'top-right', // 🔹 ubicación
    timeout: 4000,         // 🔹 se cierra en 2 segundos
    icon: 'positive',
    textColor: 'black',
    actions: [
    {
      label: 'Sí',           // texto del botón
      color: 'white',        // color del texto
      handler: () => {       // función al hacer clic
        console.log('Guardado!');
      }
    },
    {
      label: 'No',
      color: 'yellow',
      handler: () => {
        console.log('Cancelado');
      }
    }
  ]  // 🔹 icono opcional
  })
}

function mostrarTamaño() {
     console.clear()
  console.log('📏 Información de la pantalla:')
  console.log('--------------------------------')
  console.log('🖥️ Ancho:', $q.screen.width)
  console.log('🖥️ Alto:', $q.screen.height)
  console.log('📱 Nombre del tamaño:', $q.screen.name) // xs, sm, md, lg, xl
  console.log('🔹 xs:', $q.screen.xs)
  console.log('🔹 sm:', $q.screen.sm)
  console.log('🔹 md:', $q.screen.md)
  console.log('🔹 lg:', $q.screen.lg)
  console.log('🔹 xl:', $q.screen.xl)
  console.log('📉 Menor que md (lt.md):', $q.screen.lt.md)
  console.log('📈 Mayor que sm (gt.sm):', $q.screen.gt.sm)
  console.log('🪟 Orientación:', $q.screen.orientation)
  console.log('💡 Densidad de píxeles (pixelRatio):', $q.screen.pixelRatio)
  if ($q.screen.lt.md) {
    // lt.md = "menor que tamaño mediano"
    $q.notify({
      message: 'Estás en un dispositivo pequeño 📱',
      color: 'info'
    })
  } else {
    $q.notify({
      message: 'Pantalla grande 💻',
      color: 'positive'
    })
  }
}


// sessionStorage

// 🔹 Al cargar el componente, leer valor guardado (si existe)
onMounted(() => {
  const valorGuardado = $q.sessionStorage.getItem('contador')
  if (valorGuardado !== null) {
    contador.value = valorGuardado
  }
})

// 🔹 Función para aumentar y guardar
function aumentar() {
  contador.value++
  $q.sessionStorage.set('contador', contador.value)
  $q.notify({
    message: `Contador: ${contador.value}`,
    color: 'info',
    position: 'top-right'
  })
}

// 🔹 Función para reiniciar
function reiniciar() {
  contador.value = 0
  $q.sessionStorage.remove('contador')
  $q.notify({
    message: 'Contador reiniciado 🍓',
    color: 'negative'
  })
}
const nombre = ref('')
function guardar() {
  if (nombre.value.trim() === '') {
    $q.notify({ message: 'El campo está vacío 🚫', color: 'negative' })
    return
  }

  $q.notify({ message: `Guardado: ${nombre.value} ✅`, color: 'positive' })
  nombre.value = ''
}
</script>

<template>
  <q-btn color="primary" label="Agregar" @click="mostrarDialogo" />
<q-btn color="negative" label="Notificar" @click="mostrarNotificacion" />
<q-btn color="positive" label="Detectar pantalla" @click="mostrarTamaño" />
<div class="q-pa-md text-center">
    <h5>🍒 Contador de sesión: {{ contador }}</h5>
    <q-btn color="primary" label="Aumentar" @click="aumentar" class="q-ma-sm" />
    <q-btn color="negative" label="Reiniciar" @click="reiniciar" class="q-ma-sm" />
  </div>
  <q-chip clickable @click="onClick" color="primary" text-color="white" icon="shopping_cart">
      agregar al carrito
    </q-chip>
<q-input
      v-model="nombre"
      label="Escribe tu nombre"
      @keyup.enter="guardar"   
    />
</template>
