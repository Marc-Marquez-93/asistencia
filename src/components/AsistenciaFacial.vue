<script setup>
/**
 * AsistenciaFacial.vue
 * Componente para detectar rostros y marcar asistencia usando la cámara.
 */
import { ref, onMounted } from 'vue'
import * as faceapi from 'face-api.js'
import { useQuasar } from 'quasar'

const $q = useQuasar()
const videoRef = ref(null)
const nombreEstudiante = ref('')
const modo = ref('asistencia') // 'registro' o 'asistencia'
const cargando = ref(true)
const estudiantesMemoria = ref([]) // Base de datos temporal en RAM

// 1. Cargar los modelos de IA al iniciar el componente
onMounted(async () => {
  try {
    // Cargamos los modelos desde la carpeta /public/models
    await Promise.all([
      faceapi.nets.ssdMobilenetv1.loadFromUri('/models'),
      faceapi.nets.faceLandmark68.loadFromUri('/models'),
      faceapi.nets.faceRecognition.loadFromUri('/models')
    ])
    // Una vez cargados, encendemos la cámara
    startVideo()
  } catch (error) {
    console.error("Error cargando modelos. Revisa que la carpeta /public/models tenga los archivos.", error)
    $q.notify({ message: 'Error cargando IA. Revisa la consola.', color: 'negative' })
  }
})

// 2. Función para acceder a la webcam
function startVideo() {
  navigator.mediaDevices.getUserMedia({ video: {} })
    .then(stream => {
      videoRef.value.srcObject = stream
      cargando.value = false
    })
    .catch(err => {
      console.error("Error al abrir cámara:", err)
      $q.notify({ message: 'No se pudo acceder a la cámara', color: 'negative' })
    })
}

// 3. Función principal: Detectar y Procesar
async function procesarCara() {
  if (cargando.value) return

  // Detectamos una sola cara con sus puntos y su descriptor numérico
  const detection = await faceapi.detectSingleFace(videoRef.value)
    .withFaceLandmarks()
    .withFaceDescriptor()

  if (!detection) {
    $q.notify({ message: 'No veo ninguna cara, acércate más 🧐', color: 'warning', timeout: 1000 })
    return
  }

  // --- MODO REGISTRO (Guardar nueva cara) ---
  if (modo.value === 'registro') {
    if (!nombreEstudiante.value) {
      $q.notify({ message: '¡Escribe un nombre primero!', color: 'negative' })
      return
    }
    
    // Guardamos en el array
    estudiantesMemoria.value.push({
      nombre: nombreEstudiante.value,
      descriptor: detection.descriptor
    })
    
    $q.notify({ message: `Registrado: ${nombreEstudiante.value} ✅`, color: 'positive' })
    nombreEstudiante.value = '' 
    modo.value = 'asistencia' // Volvemos al modo normal
  } 
  
  // --- MODO ASISTENCIA (Verificar quién es) ---
  else {
    if (estudiantesMemoria.value.length === 0) {
      $q.notify({ message: 'No hay nadie registrado en memoria.', color: 'info' })
      return
    }

    // Preparamos el comparador (FaceMatcher) con los datos que tenemos en memoria
    const labeledDescriptors = estudiantesMemoria.value.map(e => 
      new faceapi.LabeledFaceDescriptors(e.nombre, [e.descriptor])
    )
    const faceMatcher = new faceapi.FaceMatcher(labeledDescriptors, 0.6) // 0.6 es la tolerancia
    
    // Buscamos la mejor coincidencia
    const bestMatch = faceMatcher.findBestMatch(detection.descriptor)

    if (bestMatch.label !== 'unknown') {
      $q.notify({ 
        message: `¡Asistencia OK! Hola ${bestMatch.label} 👋`, 
        color: 'positive', 
        icon: 'verified',
        position: 'center',
        timeout: 3000
      })
    } else {
      $q.notify({ message: 'Cara no reconocida 🚫', color: 'negative' })
    }
  }
}
</script>

<template>
  <div class="column flex-center q-pa-md window-height bg-grey-2">
    <h3 class="text-primary q-mb-md">Asistencia Facial SENA 🤖</h3>
    
    <div class="video-container relative-position shadow-10">
      <video ref="videoRef" autoplay muted width="640" height="480"></video>
      
      <div v-if="cargando" class="absolute-full flex flex-center bg-grey-3">
        <div class="column flex-center">
          <q-spinner color="primary" size="3em" />
          <p class="q-mt-sm">Cargando Modelos IA...</p>
        </div>
      </div>
    </div>

    <div class="q-mt-lg row q-gutter-md flex-center control-panel">
      
      <template v-if="modo === 'registro'">
        <q-input 
          v-model="nombreEstudiante" 
          label="Nombre del Estudiante" 
          filled 
          bg-color="white" 
          class="input-nombre"
          @keyup.enter="procesarCara"
        />
        <q-btn color="green" icon="save" label="Guardar Rostro" @click="procesarCara" />
        <q-btn flat color="grey" label="Cancelar" @click="modo = 'asistencia'" />
      </template>

      <template v-else>
        <q-btn size="lg" color="primary" icon="camera_alt" label="PASAR LISTA" @click="procesarCara" />
        <q-btn flat color="secondary" label="Registrar Nuevo Alumno" @click="modo = 'registro'" />
      </template>

    </div>

    <div class="q-mt-md text-caption text-grey-7">
      Alumnos en memoria RAM: {{ estudiantesMemoria.length }}
    </div>
  </div>
</template>

<style scoped>
.video-container {
  border-radius: 12px;
  overflow: hidden;
  background-color: black;
  border: 4px solid #1976D2;
}

.control-panel {
  background: white;
  padding: 20px;
  border-radius: 12px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

.input-nombre {
  min-width: 250px;
}
</style>