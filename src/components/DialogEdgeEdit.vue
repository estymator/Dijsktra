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
        Edytuj Połączenie
      </v-btn>
    </template>
    <v-card>
      <v-card-title>Wybierz połączenie</v-card-title>
      <v-divider></v-divider>
      <v-card-text style="height: 300px;">
        <v-select
        v-model="edge"
        label="Połączenie"
        :items=edges>
        </v-select>
        <v-spacer></v-spacer>
        <v-select
        v-model="weight"
        label="Waga"
        :items=[1,2,3,4,5,6,7,8,9]>
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
          @click="editEdge"
        >
          Edytuj
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
        edge: 0,
        weight:1,
      }
    },
    props:['edges'],
    methods:{
      editEdge(){
        this.dialog=false
        this.$emit('result',[this.edge,this.weight])
      }
    }
  })
</script>
