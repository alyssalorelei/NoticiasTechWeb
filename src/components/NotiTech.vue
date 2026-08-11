<template>
  <div>
    <!-- BANNER SUPERIOR -->
    <nav class="navbar navbar-expand-lg navbar-light bg_banner shadow-sm py-3 mb-5">
      <div class="container d-flex justify-content-between align-items-center">
        
        <!-- Nombre de la Aplicación -->
        <a class="navbar-brand text-primary fs-3 fw-bold text-white mb-0" href="#">
          NoticiasTech
        </a>

        <!-- Botón para Importar Noticias -->
        <button class="btn btn-primary" type="button" @click="importarNoticias">
          Importar Noticias
        </button>

      </div>
    </nav>

    <!-- CONTENIDO PRINCIPAL (Tabla y Botón de Análisis) -->
    <main class="container">
      <div class="card p-4">
        <h2 class="h5 mb-4 text-muted">Noticias Descargadas</h2>
        
        <!-- Tabla -->
        <div class="table-responsive mb-4" style="max-height: 450px; overflow-y: auto;">
            <table class="table table-striped table-hover align-middle">
                <thead class="table-light sticky-top">
                <tr>
                    <th scope="col">#</th>
                    <th scope="col">Título de la Noticia</th>
                    <th scope="col">Fuente</th>
                    <th scope="col">Fecha</th>
                    <th scope="col" class="text-center">Estatus</th>
                    <th scope="col" class="text-center">Leer</th>
                </tr>
                </thead>
                <tbody>
                <tr v-for="(noticia, index) in listaNoticias" :key="index">
                    <th scope="row">{{ index + 1 }}</th>
                    <td class="fw-medium">{{ noticia.titulo }}</td>
                    <td>{{ noticia.fuente }}</td>
                    <td class="text-muted">{{ noticia.fecha }}</td>
                    
                    <!-- COLUMNA 1: Estatus -->
                    <td class="text-center">
                    <span v-if="noticia.estatus === 'no_analizado'" class="text-muted fw-bold">--</span>
                    <span v-else-if="noticia.estatus === 'aprobado'" class="circulo-estatus bg-success" title="Cumple los filtros"></span>
                    <span v-else-if="noticia.estatus === 'rechazado'" class="circulo-estatus bg-danger" title="No cumple los filtros"></span>
                    </td>

                    <!-- COLUMNA 2: Leer -->
                    <td class="text-center">
                    <a :href="noticia.url" target="_blank" class="btn btn-sm btn-outline-primary">
                        Leer
                    </a>
                    </td>
                </tr>
                
                <!-- Mensaje si la tabla está vacía -->
                <tr v-if="listaNoticias.length === 0">
                    <td colspan="6" class="text-center py-4 text-muted">
                    No hay noticias descargadas. Haz clic en "Importar Noticias".
                    </td>
                </tr>
                </tbody>
            </table>
        </div>

        <!-- Botón Inferior para Analizar -->
        <div class="d-flex justify-content-end">
          <button class="btn btn-outline-primary btn-lg" type="button" @click="analizarNoticias" :disabled="listaNoticias.length === 0">
            Analizar Noticias
          </button>
        </div>

      </div>
    </main>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'

type EstatusNoticia = 'no_analizado' | 'aprobado' | 'rechazado';

// Definimos una interfaz (opcional, pero recomendada en TypeScript) para la estructura de las noticias
interface Noticia {
  titulo: string;
  fuente: string;
  fecha: string;
  estatus: EstatusNoticia;
  resumen: string;
  url: string;
}

const cargando = ref(false);
const analizandoTodo = ref(false);

// Datos de prueba reactivos para llenar la tabla visualmente
const listaNoticias = ref<Noticia[]>([
  
])



// Función que se ejecuta al presionar "Importar Noticias"
const importarNoticias = async (): Promise<void> => {
  cargando.value = true;
  try {

    const baseUrl = import.meta.env.VITE_API_URL;
    const response = await fetch(baseUrl + '/noticias');
    
    if (!response.ok) {
      throw new Error('Error al conectar con la API');
    }
    
    const data: Noticia[] = await response.json();
    listaNoticias.value = data;
    
    console.log("Noticias cargadas en la tabla:", listaNoticias.value);
  } catch (error) {
    console.error("Error al importar las noticias:", error);
    alert("No se pudieron cargar las noticias desde el API.");
  } finally {
    cargando.value = false;
  }
}

// Función que se ejecuta al presionar "Analizar Noticias"
const analizarNoticias= async (): Promise<void> =>{
if (listaNoticias.value.length === 0) return;

  console.log("Entrando en analizar");
  analizandoTodo.value = true;
  try {

    
    const baseUrl = import.meta.env.VITE_API_URL;
    //const response = await fetch(`${baseUrl}/noticias`);

    const response = await fetch(baseUrl +  '/analizar_noticias', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(listaNoticias.value) // Enviamos toda la tabla
    });
    console.log(response);

    if (!response.ok) throw new Error('Error al procesar el lote');

    // Recibimos la lista completa con los estatus actualizados y reemplazamos
    const data: Noticia[] = await response.json();
    listaNoticias.value = data;
    
  } catch (error) {
    console.error("Error masivo:", error);
    alert("Hubo un problema al analizar las noticias.");
  } finally {
    analizandoTodo.value = false;
  };
};

onMounted(() => {
  importarNoticias();
});
</script>

<style scoped>
/* El contenedor de la tabla usa la clase 'card' de Bootstrap, 
   heredando la sombra suave y sin bordes que definimos en estilo.css */
   
.table th {
  font-weight: 600;
  color: #4a5568;
}

.table td {
  font-size: 0.95rem;
}
</style>