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
        Dodaj Połączenie
      </v-btn>
    </template>
    <v-card>
      <v-card-title>Wybierz wierzchołki</v-card-title>
      <v-divider></v-divider>
      <v-card-text style="height: 300px;">
        <v-select
        v-model="first"
        label="Pierwszy wierzchołek"
        :items=nodes>
        </v-select>
        <v-spacer></v-spacer>
        <v-select
        v-model="second"
        label="Drugi wierzchołek"
        :items=nodes>
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
          @click="addEdge"
        >
          Dodaj
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
        first:0,
        second:0,
        weight:1,
      }
    },
    props:['nodes'],
    methods:{
      addEdge(){
        this.dialog=false
        let result=this.first+this.second
        this.$emit('result',[result,this.weight])
      }
    }
  })
</script>
