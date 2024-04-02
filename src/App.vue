<script setup>
import { computed, reactive, ref } from 'vue'
const or_model_speed = ref(1000)
const is_solving = ref(false)
const nurses = [
  { name: 'Nurse 1', id: 1 },
  { name: 'Nurse 2', id: 2 },
  { name: 'Nurse 3', id: 3 },
  { name: 'Nurse 4', id: 4 },
  { name: 'Nurse 5', id: 5 },
  { name: 'Nurse 6', id: 6 },
  { name: 'Nurse 7', id: 7 },
  { name: 'Nurse 8', id: 8 },
  { name: 'Nurse 9', id: 9 },
  { name: 'Nurse 10', id: 10 }
]

const rooms = [
  'room1',
  'room2',
  'room3',
  'room4',
  'room5',
  'room6',
  'room7',
  'room8',
  'room9',
  'room10'
]

const state = reactive({
  room1: { patient: 'A', nurse: -1 },
  room2: { patient: 'B', nurse: -1 },
  room3: { patient: 'C', nurse: -1 },
  room4: { patient: 'D', nurse: -1 },
  room5: { patient: 'E', nurse: -1 },
  room6: { patient: 'F', nurse: -1 },
  room7: { patient: 'G', nurse: -1 },
  room8: { patient: 'H', nurse: -1 },
  room9: { patient: 'I', nurse: -1 }
})

const nurse_client_count = computed(() => {
  let nurse_client_count = {}
  for (let room in state) {
    if (state[room].nurse === -1) {
      continue
    }
    if (nurse_client_count[state[room].nurse]) {
      nurse_client_count[state[room].nurse]++
    } else {
      nurse_client_count[state[room].nurse] = 1
    }
  }
  return nurse_client_count
})

const proposal = computed(() => {
  if (is_solving.value) {
    return {}
  }
  console.log(JSON.stringify(state))
  let available_nurses = nurses.filter((nurse) => {
    for (let room in state) {
      if (state[room].nurse == nurse.id) {
        return false
      }
    }
    return true
  })
  console.log(available_nurses)
  let proposal = { ...state }
  let c = 0
  for (let room in proposal) {
    proposal[room] = { ...proposal[room] }
    if (proposal[room].nurse === -1) {
      proposal[room].nurse = available_nurses[c].id
      c++
      if (c === available_nurses.length) {
        break
      }
    }
  }
  return proposal
})
let timeout = null
const on_nurse_change = (nurse_id, room) => {
  is_solving.value = true
  if (timeout) clearTimeout(timeout)
  timeout = setTimeout(() => {
    is_solving.value = false
  }, or_model_speed.value)
  state[room].nurse = nurse_id
}

const on_accept_proposal = (nurse_id, room) => {
  state[room].nurse = nurse_id
}
</script>

<template lang="pug">
  div 
  input(v-model="or_model_speed" type="range" min="100" max="10000" step="100")
  span {{ or_model_speed }} ms
  .table(v-if="state")
    .table-row.table-header
      .table-cell(style="width: 100px")
        | Room
      .table-cell(style="width: 100px")
        | Patient
      .table-cell
        | nurse
    .table-row(v-for="room in rooms" :key="room")
      .table-cell
        | {{ room }}
      .table-cell
        | {{ state[room]?.patient }}
      .table-cell(v-if="proposal" style="display:flex")
        select(v-on:change="(e) => on_nurse_change(e.target.value, room)" :value="state[room]?.nurse")
          option(:key="-1" :value="-1") Select nurse
          option(v-for="nurse in nurses" :key="nurse.id" :value="nurse.id") {{ nurse.name }}
        span(v-if="!is_solving" style="margin-left: 10px")
          span.margin {{ proposal[room]?.nurse }}
          button(v-on:click="on_accept_proposal(proposal[room]?.nurse, room)") accept
          span.margin(v-if="nurse_client_count[state[room]?.nurse] > 1", style="color: red") nurse {{ state[room]?.nurse }} as more than one patient
        

    

</template>

<style scoped>
.margin {
  margin: 10px;
}
.table {
  display: table;
  width: 100%;
  border-collapse: collapse;
}
.table-row {
  display: table-row;
}
.table-header {
  font-weight: bold;
  color: green;
}
.table-cell {
  display: table-cell;
  border: 1px solid #000;
  padding: 8px;
}
</style>
