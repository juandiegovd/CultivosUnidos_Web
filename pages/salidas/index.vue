<template>
  <v-container>
    <page-title>
      SALIDAS DE INSUMOS
    </page-title>
    <salidas/>
  </v-container>
</template>

<script>
import Salidas from "@/components/salidas/Salidas";

export default {
  name: "index",
  components:{
    "salidas":Salidas,
  }
}
</script>

<style scoped>

</style>
