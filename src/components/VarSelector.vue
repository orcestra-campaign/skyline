<script setup>
import { ref, watch, computed, defineModel } from "vue";
import * as zarr from "zarrita";
import dayjs from 'dayjs';
import utc from 'dayjs/plugin/utc';
dayjs.extend(utc);


const timeinfo = ref(null);
const currentTime = ref(null);
const currentTimeIndex = defineModel("timeIndex", 0);

const emit = defineEmits(["varSelect", "timeSelect"]);

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

function decodeTime(value, attrs) {
    const units_re = /([a-zA-Z]+) since (.+)$/;
    const [, interval, refdate] = attrs.units.match(units_re);
    const ref = dayjs.utc(refdate);
    const timepoint = ref.add(value, interval);
    return timepoint;
}

function encodeTime(value, attrs) {
    const units_re = /([a-zA-Z]+) since (.+)$/;
    const [, interval, refdate] = attrs.units.match(units_re);
    const ref = dayjs.utc(refdate);
    return value.diff(ref, interval);
}


async function getTime() {
  const root = await zarr.open(store.value, { kind: "group" });
  const arr = await zarr.open(root.resolve("time"), {kind: "array"});
  const timedata = (await zarr.get(arr, [null])).data;
  console.log("time", timedata, arr.attrs);
  console.log(decodeTime(timedata[0], arr.attrs))
  timeinfo.value = {
    data: timedata,
    attrs: arr.attrs,
  }
  setTimeIndex(timeIndex.value);
}

getVars();
getTime();

function binsearch(arr, x, start=0, end=null) {
  if (end === null) {
    end = arr.length;
  }
  if (start > end) {
    return end;
  }
  const mid = Math.floor(start + (end - start) / 2);
  if (arr[mid] == x) {
    return mid;
  } else if (arr[mid] > x) {
    return binsearch(arr, x, start, mid - 1);
  } else {
    return binsearch(arr, x, mid + 1, end);
  }
}

const timeIndex = computed(() => {
  try {
    const timeval = encodeTime(currentTime.value, timeinfo.value.attrs);
    return binsearch(timeinfo.value.data, timeval);
  } catch {
    return 0;
  }
})

const maxTimeIndex = computed(() => {
  if (timeinfo.value) {
    return timeinfo.value.data.length - 1;
  } else {
    return 0;
  }
})

function setTimeIndex(i) {
  currentTime.value = decodeTime(timeinfo.value.data[i], timeinfo.value.attrs);
  emit("timeSelect", i);
}

watch(variable, (newVariable) => {
  emit("varSelect", newVariable);
})

watch(currentTimeIndex, setTimeIndex);

</script>


<template>
  <div class="varselect">
    <select v-model="variable">
      <option disabled value="">Select Variable</option>
      <option v-for="option in variables" :key="option" :value="option">
        {{ option }}
      </option>
    </select>
    <input type="range" id="time" name="time" :min="0" :max="maxTimeIndex" v-model="currentTimeIndex" />
    {{ currentTime }}
  </div>
</template>