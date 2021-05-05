<template>
  <div v-if="login && !register" id="app">
   <div id="misGastos">
     <h1 id="titulo">Mis gastos</h1>
     <div class="mt-4">
        <p class="parrafo">Agrega un gasto a la lista</p>
     </div>
     <div >
        <button id="singIn" type="button" role="button" v-on:click='system($event)'>
          Sign in
        </button>
        <button id="logOut" type="button" role="button" v-on:click='system($event)'>
          Log Out
        </button>
     </div>
     <div class="container bg-color rounded py-3">
        <span id='show' v-bind:class="'pointer '+show" v-on:click='controlInput'></span> 
        <form id='input' v-show="input" class="w-50 m-auto">
          <div class="form-group row text-left">
            <label class="col-12 col-sm-4 col-form-label" for="name">Nombre</label>
            <input 
              v-model="nameValue" 
              class="col-12 col-sm-8 form-control" 
              type="text"
              placeholder="Nombre del gasto" id='name'>
          </div>
          <div class="form-group row text-left">
            <label class="col-12 col-sm-4 col-form-label" for="monto">Monto</label>
            <input 
              v-model="amountValue" 
              v-bind:class="'col-12 col-sm-8 form-control '+valido" 
              type="text" 
              placeholder="Monto del gasto" id='monto'>
          </div>
           <div class="form-group row text-left">
            <label class="col-12 col-sm-4 col-form-label" for="type">Type</label>
            <input 
              v-model="typeValue" 
              class="col-12 col-sm-8 form-control" 
              type="text" 
              placeholder="Tipo de gasto" id="type">
          </div>
          <button id='agregar' role="button" @click.prevent="agregarGasto">Agragar</button>
        </form>
     </div>
     <div v-show="cancel">
        <span id='cancel' class="pointer" v-on:click='cancelEdit' role="button" data-bs-toggle="tooltip" data-bs-placement="right" title="Cancel edition">X</span>
     </div>  
     <div class="container mt-5">
        <ul class="content-tabs row">
          <li class="item pointer" v-for="(gasto, index) in listaTipo" :key="index">
            <a class="item-link" :id="gasto.type" @click.prevent='selected($event,gasto.type)'>{{gasto.type}}</a>
          </li>
          <li class="item pointer">
            <span v-show="times" id='delete' class='fa fa-times' v-on:click='eliminarFiltro'></span>
          </li>
        </ul>
     </div>
     <div class="container mt-2 bg-light rounded">
        <div class="row justify-content-center mark-text titulo-seccion py-2">
          <div class="col-4 text-left">Nombre</div>
          <div class="col-2 ">Monto</div>
          <div class="col-3 ">Tipo</div>
          <div class="col-3 ">Accion</div>
        </div>
        <MisGastos 
          v-for="(gasto, index) in listaGastos"
          v-bind:key="index"
          v-bind:type='type'
          v-bind:id="gasto.id"
          v-bind:gasto="gasto"
          v-bind:indice="index"
          v-on:eliminarGasto="eliminarGasto($event)"
          v-on:editarGasto="editarGasto($event)"/>
        <div class="row total py-2">
          <div class="text-left col-4 mark-text">Total</div>
          <div class="col-4"><span>${{num}}</span></div>
        </div>
     </div>
    </div>
  </div>
  <div v-else-if="register">
    <SingIn v-bind:firebase='firebase' v-on:goBack="this.register = false"/>
  </div>
  <div v-else>
    <LoginForm v-bind:firebase='firebase' @loginApp="loginApp($event)"/>
  </div>
</template>

<script>
import firebase from 'firebase'
import 'firebase/firestore'
import SingIn from './components/SingIn.vue'
import MisGastos from './components/MisGastos.vue'
import LoginForm from './components/LoginForm.vue'

export default {
  name: 'App',
  components: {
    MisGastos,
    LoginForm,
    SingIn
  },
  data:function() {
    return {
      login: false,
      input: true,
      register: false,
      show: 'fa fa-minus-square',
      valido: '',
      nameValue: '',
      amountValue: '',
      typeValue: '',
      listaGastos: [],
      listaTipo: [],
      coleccion: {},
      type: '',
      num: 0,
      amountResult: [],
      editar: false,
      times: false,
      cancel: false,
      gastoId: '',
      updateTabs: false,
      firebase: '',
    }
  },
  methods: {
    loginApp:function(usuario) {
      let user = usuario
      this.coleccion = this.db.collection('/usuarios/'+user+'/gastos')
      this.coleccion.get()
       .then((gastos) => {
         gastos.forEach((gasto) => {
           this.listaGastos.unshift({
             id: gasto.id,
             name: gasto.data().name,
             amount: gasto.data().amount,
             type: gasto.data().type
           })
           this.num += gasto.data().amount
         })
         gastos.forEach((gasto) => {
           let type = gasto.data().type
           !(type in gastos) && (gastos[type] = true) && this.listaTipo.unshift({
             type     
           })  
         })
         this.login = true
       })
    },
    system:function(e) {
      if (e.target.id === 'singIn') {
        this.register = true
      } else if (e.target.id === 'logOut'){
        this.login = false
        this.listaGastos = []
        this.listaTipo = []
        this.num = 0
      }
    },
    controlInput:function() {
      this.input = !this.input
      this.show = this.input ? 'fa fa-minus-square' : 'fa fa-plus-square'
    },
    selected:function(e, type) {
      this.times = true
      this.num = 0
      this.type = type
      this.amountResult = this.listaGastos.filter(function(e) {
        return e.type === type
      })
      this.amountResult.forEach((e) => {
        this.num += e.amount
      })
      for (let item of this.listaTipo) {
        if (item.type === e.target.id) {
          document.getElementById(type).className = 'item-link selected'
        } else {
          document.getElementById(item.type).className= 'item-link'
        }
      }
    },
    eliminarFiltro:function() {
      this.times = false
      this.num = 0
      this.listaGastos.forEach((e) => {
        this.num += e.amount
      })
      this.type = ''
      for (let item of this.listaTipo) {
        document.getElementById(item.type).className= 'item-link'
      }
    },
    cancelEdit:function() {
      this.cancel = false
      this.editar = false
      this.nameValue = ''
      this.amountValue = ''
      this.typeValue = ''
      this.controlInput()
    },
    editarGasto:function(gastoId) { 
      this.cancel = true
      this.editar = true
      this.gastoId = gastoId
      let gasto = this.listaGastos[gastoId.indice]
      if (!this.input) {
        this.input = true 
        this.show = 'fa fa-minus-square' 
      }
      this.nameValue = gasto.name
      this.amountValue = gasto.amount
      this.typeValue = gasto.type
    },
    tabsFilter:function(gasto) {
      return this.listaGastos.filter((e) => {
        return e.type === gasto.type
      })
    },
    eliminarGasto:function(gastoId) { 
      if (window.confirm('Are you sure to remove it from the list?')) {
        let gasto = this.listaGastos[gastoId.indice]
        let tabsFilter = this.tabsFilter(gasto) 
        for (let i = 0; i < this.listaTipo.length; i++) {
          if (this.listaTipo[i].type === gasto.type && tabsFilter.length < 2) {
            this.listaTipo.splice(i, 1)
          }
        } 
        this.coleccion.doc(gastoId.id).delete()
        this.listaGastos.splice(gastoId.indice, 1) 
        this.num = this.num - gasto.amount
      } 
    },
    agregarGasto:function() {
      let objectGastos = {
        name: this.nameValue,
        amount: isNaN(this.amountValue) ? null : Number(this.amountValue),
        type: this.typeValue
      }
      if (this.editar) {
        this.num = this.num - this.listaGastos[this.gastoId.indice].amount
        let gasto = this.listaGastos[this.gastoId.indice]
        let tabsFilter = this.tabsFilter(gasto)
        for (let i = 0; i < this.listaTipo.length; i++) {
          if (this.listaTipo[i].type === gasto.type && tabsFilter.length < 2) {
            this.listaTipo.splice(i, 1)
          }
        } 
        this.coleccion.doc(this.gastoId.id).update(objectGastos)
          this.listaGastos.splice(this.gastoId.indice, 1 ,{
            id: this.gastoId.id,
            name: objectGastos.name,
            amount: objectGastos.amount,
            type: objectGastos.type
          })
        this.editar = false  
      } else {
        this.coleccion.add(objectGastos)
         .then((documento) => {
            console.log(documento)
            this.listaGastos.unshift({
              id: documento.id,
              name: objectGastos.name,
              amount: objectGastos.amount,
              type: objectGastos.type
            })
         })
         .catch((error) => {
           alert('No se pudo agregar el libro al sistema.Error: '+ error.message)
         })
      }
      this.num += objectGastos.amount
      let type = objectGastos.type
      for (let ele of this.listaTipo) {
        if (objectGastos.type === ele.type) {
          type = ''
        }  
      }
      if (type) {
        this.listaTipo.unshift({
          type: type
        })
      }
      this.nameValue = ''
      this.amountValue= ''
      this.typeValue = ''
      this.cancel = false
    }
  },
  updated:function() {
    localStorage.setItem('dataLogin', JSON.stringify(this.login))
    localStorage.setItem('dataNum', JSON.stringify(this.num))
    localStorage.setItem('dataTabs', JSON.stringify(this.listaTipo))
    localStorage.setItem('dataGastos', JSON.stringify(this.listaGastos))
  },
  mounted:function() {
    const dataLogin = JSON.parse(localStorage.getItem('dataLogin'))
    const dataGastos = JSON.parse(localStorage.getItem('dataGastos'))
    const dataNum = JSON.parse(localStorage.getItem('dataNum'))
    const dataTabs = JSON.parse(localStorage.getItem('dataTabs'))
    this.login = dataLogin
    if (dataGastos.length > 0) {
      this.listaGastos = dataGastos
    }
     if (dataTabs.length > 0) {
      this.listaTipo = dataTabs
    }
     if (dataNum > 0) {
      this.num = dataNum
    }
  },
  beforeMount:function() {
    let firebaseConfig = {
      apiKey: "AIzaSyCvWt4g1-ljgucslUxDiP_ZNcj1nAb7vIk",
      authDomain: "app-mis-gastos.firebaseapp.com",
      projectId: "app-mis-gastos",
      storageBucket: "app-mis-gastos.appspot.com",
      messagingSenderId: "821603135063",
      appId: "1:821603135063:web:64f7c50e9462fe802f3d12"
    };
    firebase.initializeApp(firebaseConfig)
    this.db = firebase.firestore()
    const settings = {timestampsInSnapshots: true}
    this.db.settings(settings)
    this.firebase = firebase
  }
}
</script>

<style>

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #33475b;
  padding: 60px 0;
  max-width: 600px;
  min-width: 150px;
  margin: 0 auto;
}
#app a {
  text-decoration: none;
  display: inline-block;
}

#titulo {
  color: #0091ae;
  font-weight: 600;
}
.parrafo {
  font-size: 20px;
}

.pointer {
  cursor: pointer;
}

.content-tabs {
  padding: 10px 0 0 0; 
  display: flex;
  background-color: #33475b;
  border-radius: 5px 5px 0 0;
}
.item {
  list-style: none;
  color: rgb(255, 255, 255);
 
}
.item-link{
  display: block;
  width: 6em;
  position: relative;
  border-bottom: 3px solid rgba(0, 0, 0, 0.05); 
  border-left: 3px solid rgba(0, 0, 0, 0.05); 
  transition: background 0.5s ease-in-out;
  color: rgb(255, 255, 255);
  font-weight: 600;
  border-radius: 5px 5px 0 0;
  padding: 3px 5px;
}
.item-link:hover{
  background-color: rgb(255, 255, 255);
  color: #33475b;
}
#delete {
  margin-left: 20px;
  color: rgb(255, 255, 255);
}
.selected {
  background-color: rgb(255, 255, 255);
  color: #33475b;
}

.bg-light {
  border: 1px solid rgba(0, 0, 0, 0.05); ;
}
.bg-color {
  background: linear-gradient(45deg, rgb(106, 120, 209), rgb(0, 164, 189));
  color: rgb(255, 255, 255);;
}

form {
  transition: transform 0.3s linear 0s, max-height 0.3s linear 0s;
}
.form-group{
  margin-top: 1rem;
}
.form-group label {
  text-align: center;
}
.form-group input, button {
  background-color: rgb(255, 255, 255);
  border: 0;
}
#agregar {
  color: #33475b;
  border-radius: 5px;
  padding: 5px 10px;
}

.titulo-seccion {
  border-bottom: 2px solid rgba(0, 0, 0, 0.05); 
}
.py {
  padding: 5px 0; 
}

.total {
  border-top: 2px solid rgba(0, 0, 0, 0.05); 
}
.total span {
  color: #0091ae;
}

.mark-text {
  font-weight: 600;
}

 #cancel {
  position: absolute;
  font-weight: 700;
  font-size: 15px;
  transition: 0.1s;
}
#cancel:hover {
  font-size: 20px;
}
@media (min-width: 576px) {
  .form-group label {
    text-align: left;
  }
}



</style>
