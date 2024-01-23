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

async function getVars() {
  const root = await zarr.open(store.value);
  console.log(root);
}
getVars();

const variable = defineModel("variable", {default: "cc", type: String});

watch(variable, (newVariable) => {
  emit("varSelect", newVariable);
})

</script>


<template>
  <div class="varselect">
    <label :for="varInput">variable:</label> <input ref="varInput" v-model="variable" />
  </div>
</template>