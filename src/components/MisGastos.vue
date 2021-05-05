<template>
  <div class="listGastos">   
    <div v-if="type === gasto.type" v-bind:amount="gasto.amount" class="row py">
      <div class="col-4 text-left">{{gasto.name}}</div>
      <div class="col-4">{{gasto.amount}}</div>
      <div class="col-2">{{gasto.type}}</div>
      <div class="col-2"> 
        <a href="#input" id='editar' 
          class="fa fa-pencil pointer mr-2" 
          v-on:click="editar({indice: indice, id: id})"></a>
        <div 
         id='eliminar' 
         class="fa fa-trash pointer" 
         v-on:click="eliminar({indice: indice, id: id})"></div> 
      </div>
    </div>
    <div v-if='!type' class="row py">
      <div class="col-4 text-left">{{gasto.name}}</div>
      <div class="col-2">{{gasto.amount}}</div>
      <div class="col-3">{{gasto.type}}</div>
      <div class="col-3"> 
        <a href="#input" id='editar' 
          class="fa fa-pencil pointer mr-2" 
          v-on:click="editar({indice: indice, id: id})"></a>
        <div 
         id='eliminar' 
         class="fa fa-trash pointer" 
         v-on:click="eliminar({indice: indice, id: id})"></div> 
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'MisGatos',
  props: ['gasto', 'id', 'indice', 'type'],
  methods: {
    eliminar:function(gastoId) {
      this.$emit('eliminarGasto', gastoId)
    },
    editar:function(gastoId) {
       this.$emit('editarGasto', gastoId)
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
.listGastos .pencil {
  display: inline-block
}
</style>
