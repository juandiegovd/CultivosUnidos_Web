<template>
  <v-container>
    <v-row>
      <v-col>
        <v-card
          elevation="2"
          outlined
          class="rounded-card"
        >
          <v-card-text>
            <v-row>
              <v-col class="text-center">
                <div class="text-h6 primary-color mr-8 mb-0" align="center"><br/>FECHA DE ENTRADA DE PRODUCTOS:</div>
              </v-col>
              <v-col align="center" class="mt-2">
                <menu-datepicker
                  :label="dateLabel"
                  @event-change-date="handlerChangeStartDate"
                >
                </menu-datepicker>
              </v-col>
            </v-row>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
    <v-row>
      <v-col>
        <tabla-productos-entrada @event-fill-supplies="handlerFillSupplies"/>
        <mensaje-confirmacion
          :dialog-confirm="dialogConfirm"
          :message="messageConfirm"
          @event-validate="handlePendingConfirm"
        >
        </mensaje-confirmacion>
        <accion-correcta
          :dialog-success= 'dialogSuccess'
          :message = 'messageSuccess'
          @event-success = "handleSuccess"
        ></accion-correcta>
      </v-col>
    </v-row>
    <v-row>
      <v-col>
        <v-card
          elevation="2"
          outlined
          class="rounded-card"
        >
          <v-card-text>
            <v-row >
              <v-col class="text-center">
                <div class="text-h6 primary-color mr-8 mb-0" align="center"><br/>TIPO DE ENTRADA DE PRODUCTOS</div>
              </v-col>
              <v-col align="center" class="mt-2">
                <v-select label="SELECCIONAR EL TIPO DE ENTRADA" :items="entryTypes" item-value="value" item-text="text" v-model="select" :rules="flowTypeValidation" v-on:change="updateType">
                </v-select>
              </v-col>
            </v-row>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
    <v-card-actions>
      <v-row align="center"
             justify="center" class="mt-3">
        <v-col class="text-right">
          <v-btn depressed color="error"  @click="$router.go(-1)">CANCELAR</v-btn>
        </v-col>
        <v-col>
          <v-btn depressed color="success" :disabled="checkData" @click="savePendingConfirm">ACEPTAR</v-btn>
        </v-col>
      </v-row>
    </v-card-actions>
  </v-container>
</template>

<script>
import MenuDatePicker from "@/components/MenuDatePicker";
import AccionCorrecta from "@/components/AccionCorrecta";
import MensajeConfirmacion from "@/components/MensajeConfirmacion";
import {mapActions, mapState} from "vuex";
import moment from "moment-timezone";
import TablaProductosEntrada from "@/components/entradasProducto/nuevo/TablaProductosEntrada";
import {flowTypeRules} from "@/helpers/validation";

export default {
  name: "NuevaEntradaProducto",
  data(){
    return{
      entryDate: null,
      select: null,
      dateLabel:"Ingresa fecha de entrada",
      dialogConfirm:false,
      messageConfirm:"¿Está seguro de realizar el registro?",
      dialogSuccess: false,
      messageSuccess:"ENTRADA DE PRODUCTOS REGISTRADA!",
      cantValidation:true,
      flowTypeValidation:flowTypeRules,
      entryTypes:[
        {
          value: "COMPRA",
          text: "POR COMPRA",
        },

        {
          value: "ASOCIACION",
          text: "POR ASOCIACION DE NUEVO PRODUCTOR AGRÍCOLA",
        },
      ],
      entries:[],
    }
  },
  async created() {
    await this.getEntryTypesToSelect();
  },
  components:{
    "menu-datepicker":MenuDatePicker,
    "tabla-productos-entrada":TablaProductosEntrada,
    "accion-correcta":AccionCorrecta,
    "mensaje-confirmacion":MensajeConfirmacion,
  },
  methods:{
    ...mapActions({
      getEntryTypes:'entradas/entry/getEntryTypes',
      registerEntry:'entradas/entry/registerEntry',
    }),

    handlerChangeStartDate(input){
      this.entryDate = input
    },

    handlerFillSupplies(input){
      this.supplies = input;
      this.cantValidation = this.supplies.length === 0;
      this.supplies.forEach(x =>{
        this.checkCants(x);
      })
    },

    handlePendingConfirm(value){
      this.dialogConfirm = false;
      if(value) this.save();
    },

    checkCants(input){
      if (input.entryCant === null)
        this.cantValidation = true;
    },

    updateType(item){
      this.$router.push({path: item.src});
    },

    async registerNewEntry(item){
      await this.registerEntry({
        communityId: 1, detailsToRegister:item.detailsToRegister,
        entryDate: item.entryDate, entryType: item.entryType, subtype:"ENTRADA_PRODUCTO", producerId:item.producerId
      });
    },

    handleSuccess(value){
      this.dialogSuccess = value;
      if(!this.dialogSuccess){
        this.$router.push('/entradasProducto')
      }
    },
    async getEntryTypesToSelect(){
      await this.getEntryTypes();
    },

    savePendingConfirm(){
      this.dialogConfirm = true;
    },

    async save(){
      const entryDTO = {};
      entryDTO.entryDate = (this.entryDate != null)? this.entryDate: moment().tz('America/Lima').format().slice(0, 10);
      entryDTO.entryType = this.select;
      entryDTO.detailsToRegister = this.supplies;
      entryDTO.producerId = 0;
      await this.registerNewEntry(entryDTO);
      this.dialogSuccess = true;
    },
  },
  computed:{
    ...mapState({
      types: state => state.entradas.entry.types,
      community: state => state.comunidad.community,
    }),

    checkData(){
      const checkSelect = this.select === null;
      return checkSelect || this.cantValidation;
    },
  },
}
</script>

<style scoped>

</style>
