<template>
  <v-container>
    <v-row>
      <v-col>
        <v-card>
          <v-tabs
            background-color="#f1f1f1"
            color="green lighten-2"
            center-active
            grow
          >
            <v-tab href="#production" @click="changeToCrops">POR PRODUCCION</v-tab>
            <v-tab href="#earnings" @click="changeToEarnings">POR GANANCIA DESEADA</v-tab>
            <v-tab-item value="production">
              <v-form
                ref="formProduction"
                v-model="isFormProdValid"
              ><v-row >
                <v-col class="text-center">
                  <div class="text-h6 primary-color mr-8 mb-0" align="center"><br/>MES DE VENTA:</div>
                </v-col>
                <v-col align="center" class="mt-2">
                  <v-select label="SELECCIONAR EL MES DE LA VENTA" class="mt-3 mr-16" :items="months" item-value="value" item-text="text" :rules="projectMonthValidation" v-model="month">
                  </v-select>
                </v-col>
              </v-row></v-form>
              <v-row>
                <v-col>
                  <tabla-productos :rowsAux="products" @event-fill-products="handlerFillProducts"/>
                </v-col>
              </v-row>
              <v-row align="center"
                     justify="center">
                <v-btn
                  color="green lighten-2"
                  depressed
                  class="mt-5 mb-10 white--text"
                  @click="projection"
                  :disabled="validateProjectionCrops"
                >
                  PROYECTAR
                </v-btn>
              </v-row>
            </v-tab-item>
            <v-tab-item value="earnings">
              <v-form
                ref="formEarnings"
                v-model="isFormEarnValid"
              ><v-row>
                <v-col class="text-center">
                  <div class="text-h6 primary-color mr-8 mb-0" align="center"><br/>INGRESAR LA GANANCIA TOTAL (S/.):</div>
                </v-col>
                <v-col>
                  <v-text-field align="center" class="mt-6 shrink prueba" v-model="total" label="Ingresar la ganancia total esperada" :rules="projectTotalValidation" outlined dense></v-text-field>
                </v-col>
              </v-row>
              <v-row>
                <v-col class="text-center">
                  <div class="text-h6 primary-color mr-8 mb-0" align="center"><br/>MES DE VENTA:</div>
                </v-col>
                <v-col align="center" class="mt-2">
                  <v-select label="SELECCIONAR EL MES DE LA VENTA" class="mt-3 mr-16" :items="months" item-value="value" item-text="text" :rules="projectMonthValidation" v-model="month">
                  </v-select>
                </v-col>
              </v-row></v-form>
              <v-row>
                <v-col>
                  <tabla-productos-ganancia :rowsAux="productsEarnings" @event-fill-earnings="handlerFillEarnings"/>
                </v-col>
              </v-row>
              <v-row align="center" justify="center">
                <span style="color: red" v-if="!subtotalValidation && productsEarnings.length !== 0"> ERROR! Suma de subtotales no coincide con la ganancia total</span>
              </v-row>
              <v-row align="center"
                     justify="center">
                <v-btn
                  color="green lighten-2"
                  class="mt-5 mb-10 white--text"
                  depressed
                  :disabled="!subtotalValidation || month === null"
                  @click="projectionEarnings"
                >
                  PROYECTAR
                </v-btn>
              </v-row>
            </v-tab-item>
          </v-tabs>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import TablaProductos from "@/components/proyecciones/produccion/TablaProductos";
import VueRouter from 'vue-router'
import {mapActions} from "vuex";
import TablaProductosGanancia from "@/components/proyecciones/ganancia/TablaProductosGanancia";
import {projectMonthRules, projectTotalRules} from "@/helpers/validation";

export default {
  name: "Proyecciones",
  data(){
    return{
      month: null,
      total: '',
      products:[],
      productsEarnings:[],
      cantValidation: true,
      subtotalValidation:false,
      cantValidationSubtotal: true,
      months:["ENERO", "FEBRERO", "MARZO", "ABRIL", "MAYO", "JUNIO", "JULIO", "AGOSTO", "SETIEMBRE", "OCTUBRE", "NOVIEMBRE", "DICIEMBRE"],
      projectMonthValidation: projectMonthRules,
      projectTotalValidation: projectTotalRules,
      subtotalAux: 0.00,
      isFormProdValid:false,
      isFormEarnValid:false,
    }
  },
  components:{
    "tabla-productos":TablaProductos,
    "tabla-productos-ganancia":TablaProductosGanancia,
  },

  watch:{
    total: function(val){
      let subtotalAux = 0.00;
      this.productsEarnings.forEach( x=>{
        subtotalAux+=x.subtotal;
      });
      this.subtotalValidation = subtotalAux === Number.parseFloat(val);
    }
  },
  methods:{
    ...mapActions({
      saveData:"proyecciones/projections/saveData",
    }),

    async projectionEarnings(){
      await this.saveData({month:this.month, productsData:this.productsEarnings});
      await this.$router.push('/proyecciones/ganancia');
    },

    async projection(){
      await this.saveData({month:this.month, productsData:this.products});
      await this.$router.push('/proyecciones/cultivo');
    },

    checkCants(input){
      const checkSacksEmpty = ''.localeCompare(input.soldSacks) === 0;
      const checkNumbers = !Number.isInteger(Number(input.soldSacks)) || !Number.isFinite(Number(input.soldSacks));
      const checkNull = input.soldSacks === null || input.soldSacks === undefined;
      if (checkSacksEmpty||checkNumbers||checkNull){
        this.cantValidation = true;
      }else{

        input.soldSacks = Number.parseInt(input.soldSacks);
      }
    },

    checkSubtotals(input){
      const checkSacksEmpty = ''.localeCompare(input.subtotal) === 0;
      const checkNumbers = !Number.isInteger(Number(input.subtotal)) || !Number.isFinite(Number(input.subtotal));
      const checkNull = input.subtotal === null || input.subtotal === undefined;
      if (checkSacksEmpty||checkNumbers||checkNull){
        this.cantValidationSubtotal = true;
      }else{

        input.subtotal = Number.parseInt(input.subtotal);
        this.subtotalAux+=input.subtotal;
      }
    },

    //handlers
    changeToEarnings(){
      this.month = null;
      this.products=[];
      if (this.$refs.formProduction !== undefined && this.$refs.formProduction !== null) this.$refs.formProduction.reset();
    },
    changeToCrops(){
      this.month = null;
      this.total = '';
      this.productsEarnings = [];
      if (this.$refs.formEarnings !== undefined && this.$refs.formEarnings !== null) this.$refs.formEarnings.reset();
    },
    handlerFillProducts(input){
      this.products = input;
      this.cantValidation = this.products.length === 0;
      this.products.forEach( x=>{
        this.checkCants(x);
      })
    },

    handlerFillEarnings(input){
      this.productsEarnings = input;
      this.subtotalAux = 0.00;
      this.cantValidationSubtotal = this.productsEarnings.length === 0;
      this.productsEarnings.forEach( x=>{
        this.checkSubtotals(x);
      })

      this.subtotalValidation = this.subtotalAux === Number.parseFloat(this.total);
    }
  },

  computed:{
    validateProjectionCrops:{
      get(){
        const invalidMonth = this.month === undefined || this.month === null;
        return invalidMonth || this.cantValidation;
      }
    }
  }
}
</script>

<style scoped>
.prueba{
  width: 400px;
}
</style>
