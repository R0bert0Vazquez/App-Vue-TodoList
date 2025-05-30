<template>
  <div class="contenedor-tareas">
    <h1>Lista de Tareas</h1>

    <div class="agregar-tarea">
      <h2>Agregar Nueva Tarea</h2>
      <input
        type="text"
        v-model="nuevaTarea"
        placeholder="Nueva tarea"
        @keyup.enter="agregarTarea"
      />
      <button @click="agregarTarea">Agregar</button>
    </div>

    <div class="lista-tareas">
      <h2>Lista de Tareas</h2>
      <div class="filtro">
        <label>
          <input type="checkbox" v-model="mostrarCompletadas" />
          Mostrar tareas completadas
        </label>
      </div>
      <table>
        <thead>
          <tr>
            <th>ID</th>
            <th>Tarea</th>
            <th>Estado</th>
            <th>Acciones</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="tarea in tareasFiltradas" :key="tarea.id">
            <td>{{ tarea.id }}</td>
            <td>{{ tarea.titulo }}</td>
            <td>
              <input type="checkbox" :checked="tarea.completada" @change="cambiarEstado(tarea)" />
            </td>
            <td>
              <button @click="eliminarTarea(tarea.id)">Eliminar</button>
            </td>
          </tr>
        </tbody>
      </table>
      <br />
      <button @click="eliminarCompletadas">Eliminar Tareas Completadas</button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'

interface Tarea {
  id: number
  titulo: string
  completada: boolean
}

const URL_API = 'http://localhost:3000/items'
const tareas = ref<Tarea[]>([])
const nuevaTarea = ref('')
const mostrarCompletadas = ref(false)

const tareasFiltradas = computed(() => {
  return tareas.value.filter((tarea) => mostrarCompletadas.value || !tarea.completada)
})

const cargarTareas = async () => {
  try {
    const respuesta = await fetch(URL_API)
    if (!respuesta.ok) throw new Error('Error al cargar las tareas')
    tareas.value = await respuesta.json()
  } catch (error) {
    console.error('Error al cargar las tareas:', error)
  }
}

const agregarTarea = async () => {
  if (nuevaTarea.value.trim() === '') return

  try {
    const nuevaTareaItem = {
      titulo: nuevaTarea.value,
      completada: false,
    }
    const respuesta = await fetch(URL_API, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(nuevaTareaItem),
    })
    if (!respuesta.ok) throw new Error('Error al agregar la tarea')
    const tareaAgregada = await respuesta.json()
    tareas.value.push(tareaAgregada)
    nuevaTarea.value = ''
  } catch (error) {
    console.error('Error al agregar la tarea:', error)
  }
}

const cambiarEstado = async (tarea: Tarea) => {
  try {
    const tareaActualizada = { ...tarea, completada: !tarea.completada }
    const respuesta = await fetch(`${URL_API}/${tarea.id}`, {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(tareaActualizada),
    })
    if (!respuesta.ok) throw new Error('Error al actualizar la tarea')
    const indice = tareas.value.findIndex((t) => t.id === tarea.id)
    if (indice !== -1) {
      tareas.value[indice] = tareaActualizada
    }
  } catch (error) {
    console.error('Error al actualizar la tarea:', error)
  }
}

const eliminarTarea = async (id: number) => {
  try {
    const respuesta = await fetch(`${URL_API}/${id}`, {
      method: 'DELETE',
    })
    if (!respuesta.ok) throw new Error('Error al eliminar la tarea')
    tareas.value = tareas.value.filter((tarea) => tarea.id !== id)
  } catch (error) {
    console.error('Error al eliminar la tarea:', error)
  }
}

const eliminarCompletadas = async () => {
  try {
    const tareasCompletadas = tareas.value.filter((tarea) => tarea.completada)
    await Promise.all(
      tareasCompletadas.map((tarea) =>
        fetch(`${URL_API}/${tarea.id}`, {
          method: 'DELETE',
        }),
      ),
    )
    tareas.value = tareas.value.filter((tarea) => !tarea.completada)
  } catch (error) {
    console.error('Error al eliminar las tareas completadas:', error)
  }
}

onMounted(() => {
  cargarTareas()
})
</script>

<style scoped>
.contenedor-tareas {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
}

.agregar-tarea {
  margin-bottom: 20px;
}

.agregar-tarea input {
  padding: 8px;
  margin-right: 10px;
}

table {
  width: 100%;
  border-collapse: collapse;
  margin: 20px 0;
}

th,
td {
  padding: 10px;
  text-align: left;
  border: 1px solid #ddd;
}

th {
  background-color: gray;
}

button {
  padding: 8px 16px;
  background-color: #4caf50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

button:hover {
  background-color: #45a049;
}

.filtro {
  margin: 10px 0;
}
</style>
