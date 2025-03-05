<script setup lang="ts">
import {onMounted, ref, render} from "vue";
import { asyncBufferFromUrl, parquetMetadata, parquetRead} from "hyparquet";
import wrapper from "@/components/scatterplot/wrapper.vue";
const TableData = ref();
const TableMetaData = ref();


onMounted(async () => {
  try {

    const res = await fetch('./public/drg_non_neurons.parquet')
    const arrayBuffer = await res.arrayBuffer()

    const url = 'http://localhost:5173/drg_non_neurons.parquet'

    await parquetRead({
      file: await asyncBufferFromUrl({url}),
      onComplete: (data) => {

        TableMetaData.value = parquetMetadata(arrayBuffer)
        TableData.value = data


      }
    })



  } catch (err) {
    console.error(err);
  }
})

</script>

<template>
    <wrapper :data="TableData" :meta-data="TableMetaData"/>
</template>

<style scoped>

</style>
