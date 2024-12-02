<template>
<h4>Asignatura <span class="name">{{ subject.short }}</span></h4>
    <table>
      <tbody>
        <tr>
          <th>Nombre</th>
          <td>{{ subject.name }} </td>
        </tr>
        <tr>
          <th>Créditos</th>
          <td>{{ subject.credits }} </td>
        </tr>
        <tr>
          <th>Cuatrimestre</th>
          <td>{{ subject.semester }} </td>
        </tr>
        <tr>
          <th>Códigos GEA</th>
          <td>{{ subject.codes.replaceAll('-', ', ') }} </td>
        </tr>
        <tr>
          <th>Grupos</th>
          <td v-if="subject.groups.length">
            {{ subject.groups.map(g => gState.resolve(g).name).join(' ') }}
          </td>
          <td v-else> (ninguno) </td>
        </tr>
      </tbody>
    </table>

    <h5>Acciones</h5>
    <div class="btn-group">
      <button @click="$emit('editSubject')" class="btn btn-outline-success">✏️</button>
      <button 
          :disabled="tieneGrupos" 
          @click="modalEliminarAsignatura = true" 
          class="btn btn-outline-danger"
          :title="tieneGrupos ? 'Elimine todos los grupos antes de borrar esta asignatura' : ''">
          🗑️
      </button>
    </div>

          <!-- Modal de confirmación -->
    <div v-if="modalEliminarAsignatura" class="modal fade show" tabindex="-1" style="display: block; background: rgba(0, 0, 0, 0.5);">
        <div class="modal-dialog">
          <div class="modal-content">
            <div class="modal-header">
              <h5 class="modal-title">Confirmar eliminación</h5>
              <button type="button" class="btn-close" @click="closeModal"></button>
            </div>
            <div class="modal-body">
              <p>¿Estás seguro de que deseas eliminar esta asignatura?</p>
            </div>
            <div class="modal-footer">
              <button type="button" class="btn btn-secondary" @click="closeModal">Cancelar</button>
              <button type="button" class="btn btn-danger" @click="confirmDelete">Borrar</button>
            </div>
          </div>
        </div>
    </div>
</template>

<script setup>
import { ref, computed } from 'vue';
import { gState } from '../state.js';

const emit = defineEmits(['editSubject', 'rmSubject']);

const props = defineProps({
  subject: Object, // Verifica que 'subject' esté definido como propiedad
});

// Computed para verificar si tiene grupos
const tieneGrupos = computed(() => props.subject.groups && props.subject.groups.length > 0);

// Modal state and functions
const modalEliminarAsignatura = ref(false); // Controla la visibilidad del modal

const closeModal = () => {
  modalEliminarAsignatura.value = false;
};

const confirmDelete = () => {
  closeModal();
  emit('rmSubject'); // Emite el evento para eliminar el grupo
};
</script>