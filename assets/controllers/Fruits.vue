<template>
  <div style="text-align: right">
    <button
        :disabled="pagination.currentPage <= 1"
        @click="pagination.currentPage--"
    >
      Prev Page
    </button>
    Page {{ pagination.currentPage }} of {{ pagination.totalPages }}
    <button
        :disabled="pagination.currentPage >= pagination.totalPages"
        @click="pagination.currentPage++">
      Next Page
    </button>
  </div>
  <div class="form-group col-md-4">
    <input v-model="search" placeholder="Search by name or family" class="form-control">
  </div>
  <div :class="['alert', response?.status === 'success'? 'alert-success' : response?.status === 'error'? 'alert-danger':'']">{{ response?response.message:'' }}</div>
  <table class="table">
    <thead>
    <tr>
      <th v-for="field in fields">
        {{ field.toUpperCase() }}
      </th>
      <th>Add to favourite</th>
    </tr>
    </thead>
    <tbody>
    <tr v-for="item in paginatedItems">
      <td v-for="field in fields" :key="field">
        <span v-if="field === 'nutritions'" v-for="(value, key) in item['nutritions']">
          <b>{{ key }}</b> : {{ value }}, <br>
        </span>
        <span v-else>
          {{item[field] }}
        </span>
      </td>
      <td><h3><a href="#" @click="addFavourite(item['id'],item['name'], $event)">+</a></h3></td>
    </tr>
    </tbody>
  </table>
</template>

<script setup>
import {computed, reactive, ref, watch} from 'vue';

const props = defineProps({
  items: {
    type: Array,
    required: true,
  },
  fields: {
    type: Array,
    required: true,
  }
});
const search = ref('');
const response = ref();
const pagination = reactive({
  currentPage: 1,
  perPage: 10,
  totalPages: computed(() =>
      Math.ceil(props.items.length / pagination.perPage)
  ),
});
const addFavourite = async (fruit_id, name, e) => {
  response.value = await fetch(
      '/add-favourite',
      {
        'method': 'POST',
        'body': JSON.stringify({'fruitId': fruit_id, 'name': name, 'action': e.target.innerHTML})
      }
  ).then
  (
      (r) => r.json()
  );
};

const paginatedItems = computed(() => {
  const { currentPage, perPage } = pagination;
  const start = (currentPage - 1) * perPage;
  const stop = start + perPage;
  if(search.value !== '') {
    return props.items.filter(
        (item) => (item['name'].toLowerCase().includes(search.value.toLowerCase())) ||
            (item['family'].toLowerCase().includes(search.value.toLowerCase()))
    );
  }
  return props.items.slice(start, stop);
});

watch(
    () => pagination.totalPages,
    () => (pagination.currentPage = 1)
);
</script>