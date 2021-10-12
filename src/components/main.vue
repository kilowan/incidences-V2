<template>
<div v-if="user">
  <div class="cabecera">
    <p class="mensaje">Bienvenido {{user.name}} {{user.surname1}} {{user.surname2}}</p>
    <div class="Logo">
      <a @click="logOut()" href="./">
        <img class="cierra" src="../shutdown.png" alt="Cerrar sesión" />
      </a>
    </div>
    <div class="opciones">
      <a href="#" @click="add('MakeIncidence')" v-if="user.permissions.includes('13')" class="link">Crear parte</a>
      <a href="#" @click="add('incidences')" v-if="incidencesCount >0" class="link">Ver partes</a>
      <a href="#" @click="add('statistics')" v-if="user.permissions.includes('2')" class="link" >Estadísticas</a> 
      <a href="#" @click="add('employeeList')" v-if="user.permissions.includes('16')" class="link">Lista empleados</a>
      <a href="#" @click="add('user_info')" class="link" >Datos personales</a>
    </div>
  </div>
    <div v-if="check('MakeIncidence')" class="cuerpo">
      <make-incidence v-if="user" :user="user" @closeForm="mod='Main'" class="mensaje"/>
    </div>
    <div v-else-if="check('user_info')" class="cuerpo">
      <user-info  v-if="user" :username="username" @reloadUser="reloadUser($event)"/>
    </div>
    <div v-else-if="check('statistics')" class="cuerpo">
      <statistics  v-if="user" :user="user"/>
    </div>
    <div v-else-if="check('employeeList')" class="cuerpo">
      <employee-list  v-if="user" :user="user" :incidences="incidences"/>
    </div>
    <div v-else-if="check('incidences')" class="cuerpo">
      <incidences 
      v-if="user" 
      :user="user"
      :reload="reload"
      :incidences="incidences"
      @linked="reload=false"
      @reload="reloading()"
      />
    </div>
    <div v-else class="cuerpo">
      <p>Not working</p>
    </div>

    <div class="Pie">
      <p>Trabajo realizado por Jose Javier Valero Fuentes y Juan Francisco Navarro Ramiro para el curso de ASIR 2º X migrado a Vue.js</p>
    </div>
  </div>
</template>

<script>

import axios from 'axios';

export default {
  name: 'main',
  props:{
    /*message: {
      type: String,
      required: false
    },*/
  },
  components: {
    //axios
  },
  data:function()
  {
    return {
      page: 'Login',
      mod: 'Main',
      user: undefined,
      incidences: undefined,
      incidencesCount: 0,
      reload: false,
    }
  },
 methods: {
    check: function(data)
    {
      return this.mod == data? true: false;
    },
    reloading: function()
    {
        axios.get("http://localhost:8082/newMenu.php?funcion=getAllincidences")
        .then( datas => {
          this.incidences = datas.data;
          this.reload = true;
        });
    },
    logedIn: function(data)
    {
      axios.get("http://localhost:8082/employee.php?funcion=getEmployeeByUsername&username="+ data)
      .then( datas => {
        this.user = datas.data;
        this.username = data.username;
        this.page = 'Menu';
        axios.get("http://localhost:8082/newMenu.php?funcion=getAllincidences")
        .then( datas => {
          this.incidences = datas.data;
          this.showIncidences();
        });
      });
    },
    reloadUser: function(data)
    {
      axios.get("http://localhost:8082/employee.php?funcion=getEmployeeByUsername&username="+ data)
      .then( datas => {
        this.user = datas.data;
      });
    },
    add:function(data)
    {
      if (data == 'incidences' && this.mod =='incidences') {
        this.reloading();
      }
      this.mod = data;
    },

    showIncidences: function()
    {
      let new_array = undefined;
      if(this.user.permissions.includes("7") && this.user.permissions.includes("8") && this.user.permissions.includes("9"))
      {
          new_array = this.incidences.filter(array => {
              return (array.owner.id == this.user.id && array.state != 5);
          });
          this.incidencesCount = new_array.length;
      }
      else if (this.user.permissions.includes("3") && this.user.permissions.includes("4") && this.user.permissions.includes("5"))
      {
          new_array = this.incidences.filter(array => {
              return (array.solver.id == this.user.id || array.state == 1);
          });
          this.incidencesCount = new_array.length;
      }
      else if (this.user.permissions.includes("6") && this.user.permissions.includes("7") && this.user.permissions.includes("8") && this.user.permissions.includes("9") && this.user.permissions.includes("10") && this.user.permissions.includes("11") && this.user.permissions.includes("12")) 
      {
          new_array = this.incidences.filter(array => {
              return (array.solver.id == this.user.id || (array.state == 1 || array.state == 2 || array.state == 3 || array.state == 4) || array.owner.id == this.user.id);
          });
          this.incidencesCount = new_array.length;
          this.page = 'Menu';
      }
    },
    logOut:function()
    {
      this.page = 'Login';
      this.mod = 'Main';
      this.form = {
        username: undefined,
        pass: undefined,
      };
      this.user = undefined;
      this.incidences = undefined;
      this.incidencesCount = 0;
    }
  },
  mounted(){
    debugger; //eslint-disable-line no-debugger
    if(this.$route.params.username) this.logedIn(this.$route.params.username);
  }
}
</script>
<style>
.cabecera
{
	border: 2px solid black;
  background-color: #333;
	width: 100%;
	height: 12%;
	left: 0;
	top: 0;
	position: fixed;
	color: white;
	overflow: hidden;
}
</style>
