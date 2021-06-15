<template>
    <div class="dijkstra">
        
        <v-btn gray class="edit_btn"
        >
            Graf
        </v-btn>
         <v-icon
          class="mr-2"
          @click="editItem(item)"
        >
         mdi-pencil
        </v-icon>
        {{graf}}
    </div>
</template>

<script>
export default {
    name: 'Dijkstra',
    data(){
        return{
            graf: "elo"
        }
    }
}

</script>

<style>
    
</style>
