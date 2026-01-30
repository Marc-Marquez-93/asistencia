<script setup>
/**
 * AsistenciaFacial.vue (Versión FINAL VISUAL)
 * - Nombres de modelos corregidos (con 'Net').
 * - Dibuja el cuadro y los puntos en la cara.
 */
import { ref, onMounted } from 'vue'
import { useQuasar } from 'quasar'

const $q = useQuasar()
const videoRef = ref(null)
const canvasRef = ref(null) // <--- Referencia para el dibujo
const nombreEstudiante = ref('')
const modo = ref('asistencia')
const cargando = ref(true)
const estudiantesMemoria = ref([]) 

// Carga del script local
const cargarScriptLocal = () => {
  return new Promise((resolve, reject) => {
    if (window.faceapi) return resolve() 
    const script = document.createElement('script')
    script.src = '/face-api.min.js' 
    script.defer = true
    script.onload = resolve
    script.onerror = () => reject(new Error("No encontré face-api.min.js"))
    document.head.appendChild(script)
  })
}

onMounted(async () => {
  try {
    console.log("Iniciando Sistema...")
    await cargarScriptLocal()
    const faceapi = window.faceapi

    console.log("Cargando Modelos...")
    await Promise.all([
      // 1. Detector
      faceapi.nets.ssdMobilenetv1.loadFromUri('/models'),
      // 2. Puntos (CORREGIDO: faceLandmark68Net)
      faceapi.nets.faceLandmark68Net.loadFromUri('/models'),
      // 3. Reconocimiento (CORREGIDO: faceRecognitionNet)
      faceapi.nets.faceRecognitionNet.loadFromUri('/models')
    ])

    console.log("✅ Modelos cargados.")
    startVideo()

  } catch (error) {
    console.error("Error:", error)
    $q.notify({ message: `Error: ${error.message}`, color: 'negative' })
  }
})

function startVideo() {
  navigator.mediaDevices.getUserMedia({ video: {} })
    .then(stream => {
      videoRef.value.srcObject = stream
      cargando.value = false
    })
    .catch(err => {
      console.error(err)
      $q.notify({ message: 'Sin cámara', color: 'negative' })
    })
}

async function procesarCara() {
  if (cargando.value) return
  const faceapi = window.faceapi 

  // 1. DETECCIÓN
  const detection = await faceapi.detectSingleFace(videoRef.value)
    .withFaceLandmarks()
    .withFaceDescriptor()

  if (!detection) {
    // Si no hay cara, limpiamos el dibujo anterior
    const canvas = canvasRef.value
    const ctx = canvas.getContext('2d')
    ctx.clearRect(0, 0, canvas.width, canvas.height)
    
    $q.notify({ message: 'No veo cara 🧐', color: 'warning', timeout: 500 })
    return
  }

  // --- DIBUJAR EN PANTALLA (LO NUEVO) ---
  const dims = faceapi.matchDimensions(canvasRef.value, videoRef.value, true)
  const resizedResult = faceapi.resizeResults(detection, dims)
  
  // Dibujar cuadro y puntos
  faceapi.draw.drawDetections(canvasRef.value, resizedResult)
  faceapi.draw.drawFaceLandmarks(canvasRef.value, resizedResult)

  // --- LÓGICA DE NEGOCIO ---
  if (modo.value === 'registro') {
    if (!nombreEstudiante.value) return $q.notify({ message: 'Falta nombre', color: 'negative' })
    
    estudiantesMemoria.value.push({
      nombre: nombreEstudiante.value,
      descriptor: detection.descriptor
    })
    
    $q.notify({ message: `Registrado: ${nombreEstudiante.value}`, color: 'positive' })
    nombreEstudiante.value = '' 
    modo.value = 'asistencia'
  } 
  else {
    if (estudiantesMemoria.value.length === 0) return $q.notify({ message: 'Base vacía', color: 'info' })

    const matcher = new faceapi.FaceMatcher(
      estudiantesMemoria.value.map(e => new faceapi.LabeledFaceDescriptors(e.nombre, [e.descriptor])),
      0.6
    )
    const bestMatch = matcher.findBestMatch(detection.descriptor)

    if (bestMatch.label !== 'unknown') {
      // Opcional: Dibujar el nombre en el canvas también
      const box = resizedResult.detection.box
      const drawBox = new faceapi.draw.DrawBox(box, { label: bestMatch.label })
      drawBox.draw(canvasRef.value)

      $q.notify({ message: `¡Hola ${bestMatch.label}!`, color: 'positive', icon: 'verified' })
    } else {
      $q.notify({ message: 'No te conozco', color: 'negative' })
    }
  }
}
</script>

<template>
  <div class="column flex-center q-pa-md window-height bg-grey-2">
    <h3 class="text-primary">Biometría Visual 👁️</h3>
    
    <div class="video-container relative-position shadow-10">
      
      <video ref="videoRef" autoplay muted width="640" height="480"></video>
      
      <canvas ref="canvasRef" class="absolute-top-left"></canvas>

      <div v-if="cargando" class="absolute-full flex flex-center bg-grey-3">
        <q-spinner color="primary" size="3em" />
      </div>
    </div>

    <div class="q-mt-lg row q-gutter-md flex-center">
      <template v-if="modo === 'registro'">
        <q-input v-model="nombreEstudiante" label="Nombre" filled bg-color="white" @keyup.enter="procesarCara"/>
        <q-btn color="green" icon="save" label="Guardar" @click="procesarCara" />
        <q-btn flat color="grey" label="Cancelar" @click="modo = 'asistencia'" />
      </template>
      <template v-else>
        <q-btn size="lg" color="primary" icon="camera_alt" label="ESCANEAR" @click="procesarCara" />
        <q-btn flat color="secondary" label="Nuevo" @click="modo = 'registro'" />
      </template>
    </div>
  </div>
</template>

<style scoped>
.video-container {
  border-radius: 12px;
  overflow: hidden;
  background-color: #000;
  border: 4px solid #1976D2;
  /* Importante: fijar el tamaño del contenedor al tamaño del video */
  width: 640px; 
  height: 480px;
}
</style>