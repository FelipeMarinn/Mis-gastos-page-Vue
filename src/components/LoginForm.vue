<template>
  <div class="login">
    <div class='container border rounded' style='width:350px'>
        <div class='row'>
          <div class='col-12 text-center'><h3>Ingreso al Sistema</h3></div>
        </div>
        <div class="form-group mt-4">
            <label for="email" class="cols-sm-2 control-label">Correo Electrónico</label>
            <div class="cols-sm-10">
              <div class="input-group">
                <span class="pt-2 px-3 input-group-addon form-color">
                  <i class="fa fa-envelope fa" aria-hidden="true"></i>
                </span>
                <input v-model='email' type="text" class="form-control" placeholder='Ingrese el Correo Electrónico'/>
              </div>
            </div>
        </div>
        <div class="form-group">
            <label for="password" class="cols-sm-2 control-label">Contraseña</label>
            <div class="cols-sm-10">
              <div class="input-group">
                <span class="pt-2 px-3 input-group-addon form-color">
                  <i class="fa fa-lock fa-lg" aria-hidden="true"></i>
                </span>
                <input v-model='password' type="password" class="form-control" placeholder='Ingrese la contraseña'/>
              </div>
            </div>
          </div>
        <div class="form-group text-center">
            <button type="button" class="btn form-color" @click="ingresar">Ingresar</button>
        </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'LoginForm',
  props: ['firebase'],
  data:function() {
    return {
      email: '',
      password: ''
    }
  },
  mounted:function() {
    this.email = 'estudiante@nextu.com'
    this.password = '12345678'
  },
  methods: {
    ingresar:function() {
      this.firebase.auth().signInWithEmailAndPassword(this.email, this.password)
       .then((response) => {
         this.$emit('loginApp',response.user.email)
       })
       .catch((error) => {
         console.log('Error: '+ error.code + ' - ' + error.message);
       })
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
.login{
  height: 100vh;
  display: flex;
  color: #33475b;
  align-items: center;
}
.form-color{
   background-color: #0091ae;
}
</style>
