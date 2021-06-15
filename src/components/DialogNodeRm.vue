<template>
   <v-dialog
    v-model="dialog"
    scrollable
    max-width="300px"
  >
    <template v-slot:activator="{ on, attrs }">
      <v-btn
      class="btn_main"
        dark
        v-bind="attrs"
        v-on="on"
      >
        Usuń Węzeł
      </v-btn>
    </template>
    <v-card>
      <v-card-title>Wybierz Węzeł</v-card-title>
      <v-divider></v-divider>
      <v-card-text style="height: 300px;">
        <v-select
        v-model="result"
        label="Węzeł"
        :items=nodes>
        </v-select>
      </v-card-text>
      <v-divider></v-divider>
      <v-card-actions>
        <v-btn
          color="blue darken-1"
          text
          @click="dialog = false"
        >
          Anuluj
        </v-btn>
        <v-btn
          color="blue darken-1"
          text
          @click="removeNode"
        >
          Usuń
        </v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>
    
<script>
import Vue from 'vue'
  export default Vue.extend({
    data:function(){
      return{
        dialogm1: '',
        dialog: false,
        result:0,
      }
    },
    props:['nodes'],
    methods:{
      removeNode(){
        this.dialog=false
        this.$emit('result',this.result)
      }
    }
  })
</script>
