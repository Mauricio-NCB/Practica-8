<template>
  <h4>Grupo <span class="name">{{ group.niceName }}</span></h4>
  <table>
    <tbody>
      <tr>
        <th>Asignatura</th>
        <td>{{ subject.name }}</td>
      </tr>
      <tr>
        <th>Créditos</th>
        <td>{{ group.credits }}</td>
      </tr>
      <tr>
        <th>Cuatrimestre</th>
        <td>{{ subject.semester }}</td>
      </tr>
      <tr>
        <th>Profesor</th>
        <td v-if="teacher">
          {{ teacher.userName }} ({{ teacher.firstName }} {{ teacher.lastName}})
        </td>
        <td v-else> (ninguno) </td>
      </tr>
    </tbody>
  </table>

  <SortableGrid :data="addSlotCols(slots)" :columns="slotColumns" 
    :filter="{ all: '', fields: [] }" v-model:sorter="sorter" />
  <TimeTable :slots="slots" />

  <h5>Acciones</h5>
  <div class="btn-group">
    <button @click="$emit('editGroup')" class="btn btn-outline-success">✏️</button>
    <button @click="modalEliminarGrupo = true" class="btn btn-outline-danger">🗑️</button>
  </div>

  <!-- Modal de confirmación -->
  <div v-if="modalEliminarGrupo" class="modal fade show" tabindex="-1" style="display: block; background: rgba(0, 0, 0, 0.5);">
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title">Confirmar eliminación</h5>
            <button type="button" class="btn-close" @click="closeModal"></button>
          </div>
          <div class="modal-body">
            <p>¿Estás seguro de que deseas eliminar este grupo?</p>
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
import SortableGrid from './SortableGrid.vue';
import TimeTable from './TimeTable.vue';

import { gState, weekDayNames } from '../state.js';
import { ref, computed } from 'vue'

const emit = defineEmits(['editGroup', 'rmGroup'])

const props = defineProps({
  group: Object // see definition of Group in ../model.js
})

const subject = computed(() => gState.resolve(props.group.subjectId))
const teacher = computed(() => props.group.teacherId ? gState.resolve(props.group.teacherId) : '')

let sorter = ref([{key: "weekDay", order: 1}])
const slots = computed(() => props.group.slots.map(s => gState.resolve(s)))

const slotColumns = [
    { key: 'location', display: 'Lugar', type: 'String' },
    {
        key: 'weekDay', display: 'Día', type: 'Enum',
        values: gState.model.WeekDay,
        displayValues: weekDayNames
    },
    { key: 'niceStart', display: 'Inicio', type: 'String' },
    { key: 'niceEnd', display: 'Final', type: 'String' },
]

const addSlotCols = (ss) => {
    for (let s of ss) {
        s.niceStart = formatTime(s.startTime)
        s.niceEnd = formatTime(s.endTime)
        s.niceTeacher = s.teacherId >= 0 ? gState.resolve(s.teacherId).userName : ''
    }
    return ss;
}
// 1450 => 14:50
const formatTime = t => `${Math.floor(t / 100)}:` + `0${t % 100}`.slice(-2)

// Modal state and functions
const modalEliminarGrupo = ref(false) // Controla la visibilidad del modal

const closeModal = () => {
  modalEliminarGrupo.value = false;
};

const confirmDelete = () => {
  closeModal();
  emit('rmGroup'); // Emite el evento para eliminar el grupo
};
</script>

<style scoped>
.dayOfWeek {
  display: inline-block;
  width: 4em;
}

.slot {
  display: block;
}
</style>