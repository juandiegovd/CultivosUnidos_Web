<template>
  <v-data-table
    :headers="headers"
    :items="details"
    :loading="loading"
    :items-per-page="5"
    loading-text="Cargando... Espere por favor"
  >
    <template v-slot:item.actions="{ item }">
      <v-btn
        color="green lighten-2"
        class="ma-0 white--text"
        @click="freeHectares(item)"
        v-if="!item.isFreeHectares"
      >
        LIBERAR HECTAREAS
      </v-btn>
      <v-btn
        color="green lighten-2"
        depressed
        disabled
        class="ma-0 white--text"
        v-else
      >
        LIBERADO
      </v-btn>
    </template>
  </v-data-table>
</template>

<script>
import {mapState} from "vuex";

export default {
  name: "TablaDetalle",
  props:{
    state:{
      type:String,
      default:'',
    },
    details:{
      type:Array,
      default:[],
    },
    loading:{
      type:Boolean,
      default: false,
    },
  },
  data(){
    return{
      headers1: [
        {
          text: "Productor agricola",
          value: "producerName",
          sortable: false,
          width: "500px",
        },
        {
          text: "DNI",
          value: "producerDNI",
          align: "center"
        },
        {
          text:"Hectareas ocupadas",
          value:"producerHectares",
          align: "center"
        },
        {
          text:"¿Liberar hectáreas?",
          value:"actions",
          align: "center"
        }
      ],
      headers2: [
        {
          text: "Productor agricola",
          value: "producerName",
          sortable: false,
          width: "500px",
        },
        {
          text: "DNI",
          value: "producerDNI",
          align: "center"
        },
        {
          text:"Hectareas ocupadas",
          value:"producerHectares",
          align: "center"
        },
      ],
    }
  },
  methods:{
    async freeHectares(item){
      this.$emit('event-free-hectares',item.scheduleDetailId);
    }
  },
  computed:{
    ...mapState({
      role: state => state.login.login.role,
    }),
    headers:{
      get(){
        if (this.role === "SUPERVISOR" && this.state === "EN PROCESO") return this.headers1;
        return this.headers2;
      }
    }
  },
}
</script>

<style scoped>

</style>
