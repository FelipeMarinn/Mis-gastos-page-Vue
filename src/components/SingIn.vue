<template>
  <div class="singIn">
    <div>
      <button class="back">
        <a href="" v-on:click="goBack">Go back</a>
      </button>
    </div>
    <div v-if="!register" id='form' class="container rounded border pt-3" style='width:400px'>
      <div id='titulo' class="row">
         <div class='col-12 text-center'><h3>Registro del Sistema</h3></div>
      </div>					
      <div class="form-group ">
        <label for="name" class="cols-sm-2 control-label">Your Name</label>
        <span ref="errorName" class="text-danger ml-4"></span>
        <div class="cols-sm-10">
          <div class="input-group">
            <span class="pt-2 px-3 input-group-addon bg-info"><i class="fa fa-user fa" aria-hidden="true"></i></span>
            <input v-model="name" 
             ref="name"
             type="text" 
             class="form-control" 
             name="name" id="name" 
             v-on:keyup.enter='changeFocus($event)'/>
          </div>
      </div>
      <div class="form-group">
        <label for="email" class="cols-sm-2 control-label">Choose a username</label>
        <span ref="errorUser" class="text-danger ml-4"></span>
        <div class="cols-sm-10">
          <div class="input-group">
            <span class="pt-2 px-3 input-group-addon bg-info"><i class="fa fa-envelope fa" aria-hidden="true"></i></span>
            <input type="text" 
             ref="user"
             class="form-control" 
             name="user" id="user" 
             v-model='user'
             v-on:keyup.enter='changeFocus($event)'/>
          </div>
        </div>
      </div>

      <div class="form-group">
        <label for="username" class="cols-sm-2 control-label">Email</label>
           <span ref="errorEmail" class="text-danger ml-4"></span>
        <div class="cols-sm-10">
          <div class="input-group">
            <span class="pt-2 px-3 input-group-addon bg-info"><i class="fa fa-users fa" aria-hidden="true"></i></span>
            <input type="text" 
             ref="email"
             class="form-control" 
             name="email" id="email" 
             v-model="email"
             v-on:keyup.enter='changeFocus($event)'/>
          </div>
        </div>
      </div>

      <div class="form-group">
        <label for="password" class="cols-sm-2 control-label">Choose Password</label>
        <span ref="errorPass" class="text-danger ml-4"></span>
        <div class="cols-sm-10">
          <div class="input-group">
            <span class="pt-2 px-3 input-group-addon bg-info"><i class="fa fa-lock fa-lg" aria-hidden="true"></i></span>
            <input type="password" 
             ref="pass"
             class="form-control" 
             name="password" id="password"  
             v-model="pass" 
             v-on:keyup.enter='changeFocus($event)'/>
          </div>
        </div>
      </div>

      <div class="form-group">
        <label for="confirm" class="cols-sm-2 control-label">confirm Password</label>
        <span ref="errorConfirm" class="text-danger ml-4"></span>
        <div class="cols-sm-10">
          <div class="input-group">
            <span class="pt-2 px-3 input-group-addon bg-info"><i class="fa fa-lock fa-lg" aria-hidden="true"></i></span>
            <input type="password"
             ref="confirm" 
             class="form-control" 
             name="confirm" id="confirm"  
             v-model="confirm"
             v-on:keyup.enter='changeFocus($event)'/>
          </div>
        </div>
      </div>

      <div class="form-group text-center">
        <button type="button" 
                class="btn login-button"
                @click='validateLogin($event)'>Registro</button>
      </div>
      </div>
    </div>
    <div v-else class="container">
      <div class='row successfully'>
        <h2 class='col-12 text-center'>Registro satisfactorio en el sistema</h2>
        <h5 class='col-12 text-center'>Muchas gracias por crear su registro <span>{{name}}</span></h5>
      </div>
    </div>
  </div>
</template>

<script>

export default {
  name: 'SingIn',
  props: ['firebase'],
  data: function () {
    return {
      email:'',
      name:'',
      user: '',
      pass:'',
      confirm: '',
      register:false,
    }
  },
  methods: {
    goBack:function() {
      this.$emit('goBack')
    },
    changeFocus:function(e) {
      if (e.target.id==='name'){
        this.$refs.user.focus();
      }
      else if (e.target.id==='user'){
        this.$refs.email.focus();
      }
      else if (e.target.id==='email'){
        this.$refs.pass.focus();
      }
      else if (e.target.id==='password'){
        this.$refs.confirm.focus();
      } else {
        this.validateLogin()
      }
    },
    manejaChange:function(e){
      if (e.target.id==='pdEmail'){
        this.valid = this.emailValido(this.pdEmail) ?
          'is-valid' : 'is-invalid';
      }
    },
    deleteError:function() {
      let errores = document.getElementsByClassName('text-danger')
      for (let e = 0; e < errores.length; e++) {
          errores[e].innerHTML = ""
      }
    },
    validateLogin:function(){
      this.deleteError()

      if (this.vacio(this.name)) {
        this.$refs.errorName.innerText = 'Name is required.'
        return false
      }
      if (this.vacio(this.user)) {
        this.$refs.errorUser.innerText = 'Username is required.'
        return false
      }
      if (!this.emailValido(this.email)) {
        this.$refs.errorEmail.innerText = 'invalid field'
        return false
      }
      if (!this.passValido(this.pass)) {
        this.$refs.errorPass.innerText = 'must have a minimum of six characters'
        return false
      }
      if (!this.passValido(this.confirm) || this.confirm != this.pass) {
        this.$refs.errorConfirm.innerText = 'password does not match'
        return false
      }
      this.firebase.auth().createUserWithEmailAndPassword(this.email, this.pass)
      .then((response) => {
        this.email = response.user.email
        this.register = true;
        this.$emit('fullRecord',response.user.email)
      })
      .catch(function(error) {
        let errorCode = error.code;
        let errorMessage = error.message;
        alert('Error: '+errorCode + ' - '+errorMessage)
      })
    },
    emailValido:function(email){
      const reEmail=/^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$/;
      return reEmail.test(email);
    },
    passValido:function(pass){
      const rePass=/.{6,100}/;
      return rePass.test(pass);
    },
    vacio:function(campo){
      return (campo === '');
    },
  }
}
</script>

<style scoped>
#form {
  color: #33475b;
  font-weight: 700;
}
#titulo {
  color: #33475b;
}
.singIn{
  height: 100vh;
  display: flex;
  color: #33475b;
  align-items: center;
}
label {
    opacity: 0.9;
}
#form input {
    background-color: #0091ae34;
    opacity: 0.5;
    color: rgb(17, 16, 16);
    font-weight: 600;
}
.input-group span {
    opacity: 0.8;
}
.login-button,
.input-group span {
    color: #33475b;
    font-weight: 700;
    background: #0091ae;
}

#form input:hover {
  opacity: 1;
}

.back {
  position: absolute;
  top: 15px;
  left: 20px;
}
button a {
  text-decoration: none;
  color: #0091ae;
  font-size: 20px;
}
button:after{
  position: absolute;
  content: '';
  width: 0;
  height: 2px;
  background-color: #0091ae;
  bottom: 0px;
  left: 0;
  transition: all ease-in 0.25s;
}
button:hover:after{
  width: 100%;
}

.form-group span {
     font-weight: 500;
}

.successfully span {
  color:  #0091ae;
}

</style>