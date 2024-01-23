<script setup>
import { ref, watch, computed } from "vue";
import * as zarr from "zarrita";

const varInput = ref(null);
const emit = defineEmits(["varSelect"]);

const props = defineProps({
  store: {
    type: String,
    required: true
  },
})

const store = computed(() => new zarr.FetchStore(props.store));


const variable = defineModel("variable", {default: "", type: String});
const variables = ref([]);

async function getVars() {
  const response = await fetch(props.store + "/.zmetadata");
  const zmetadata = await response.json();
  const vars = Object.entries(zmetadata.metadata)
    .filter(([key, value]) => {
      return key.endsWith("/.zattrs") &&
             value._ARRAY_DIMENSIONS[0] == "time" &&
             value._ARRAY_DIMENSIONS[1] == "level" &&
             value._ARRAY_DIMENSIONS[2] == "cell";
      })
    .map(([key, _]) => key.split("/")[0]);
  variables.value = vars;
  return vars;
}
getVars();

watch(variable, (newVariable) => {
  emit("varSelect", newVariable);
})

</script>


<template>
  <div class="varselect">
    <select v-model="variable">
      <option disabled value="">Select Variable</option>
      <option v-for="option in variables" :key="option" :value="option">
        {{ option }}
      </option>
    </select>
  </div>
</template>