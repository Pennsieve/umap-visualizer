<script setup lang="ts">
import {onMounted, ref, render} from "vue";
import {asyncBufferFromUrl, parquetMetadata, parquetRead, parquetSchema} from "hyparquet";
import wrapper from "@/components/scatterplot/wrapper.vue";
import {getColumnRange} from "hyparquet/types/column";
const TableData = ref();
const TableMetaData = ref();


onMounted(async () => {
  try {

    const res = await fetch('/DRG_nonneurons_release.parquet')
    const arrayBuffer = await res.arrayBuffer()

    const url = 'http://localhost:5173/DRG_nonneurons_release.parquet'

    await parquetRead({
      file: await asyncBufferFromUrl({url}),
      onComplete: (data) => {

        TableMetaData.value = parquetMetadata(arrayBuffer)
        TableData.value = data
        // const schema = parquetSchema(TableMetaData.value)



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
