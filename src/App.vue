<template>
  <lx-empty>
    <login v-if="page == 'Login'" :message="message" id="Login" @logedIn="logedIn($event)"></login>
    <!-- Menu -->
    <div v-if="page == 'Menu'" id="Menu">
      <div class="cabecera">
        <p class="mensaje">Bienvenido {{user.name}} {{user.surname1}} {{user.surname2}}</p>
        <div class="Logo">
          <a @click="logOut()" href="./">
            <img class="cierra" src="./shutdown.png" alt="Cerrar sesión" />
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
          <p>{{ message }}</p>
        </div>
    </div>
  </lx-empty>
</template>

<script>
import makeIncidence from './components/makeIncidence.vue';
import axios from 'axios';
import userInfo from './components/userInfo.vue';
import login from './components/Login.vue';
import statistics from './components/statistics.vue';
import employeeList from './components/employeeList.vue';
import incidences from './components/incidences.vue';

const message = 'Trabajo realizado por Jose Javier Valero Fuentes y Juan Francisco Navarro Ramiro para el curso de ASIR 2º X migrado a Vue.js';
export default {
  name: 'App',
  components: {
    login,
    makeIncidence,
    userInfo,
    statistics,
    employeeList,
    incidences,
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
      message: message,
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
      axios.get("http://localhost:8082/employee.php?funcion=getEmployeeByUsername&username="+ data.username)
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
  mounted:function()
  {}
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
body
{
	background:url("fondo.gif");
	font-size: 100%;
	font-family: sans-serif;
}
.mensaje
{
	text-align: center;
  font-size: 1.4em;
  font-weight: bold;
	color: white;
	top: 5%;
	width: 40%;
	left: 30%;
	position: relative;
}
.Logo
{
	text-align: right;
	right: 5%;
	top: 10%;
	position: absolute;
}
img.cierra
{
	width: 40px; height: 40px;
}
.login
{
  font-size: 0.9em;
  right: 1%;
  top: 9%;
  position: fixed;
}
.opciones
{
	text-align: center;
	font-size: 0.9em;
	bottom: 0;
	width: 60%;
	left: 20%;
	position: absolute;
}
.link
{
	color: white;
  padding-right: 6px;
}
.brand
{
  color: white;
  text-align: left;
}
.input
{
  margin-right: 6px;
}
.welcome
{
  top: 2%;
  left: 40%;
  position: fixed;
}
.cuerpo
{
	top: 12%;
	left: 0%;
	width: 100%;
	bottom: 10%;
	position: fixed;
	overflow: auto;
}
table
{
	box-shadow: 5px 5px 10px #999;
	border: 1px solid white;
    background: white;
	left: 10%;
	width: 80%;
	position: relative;
	border-spacing: 0px;
}
th
{
	border: 1px solid white;
    color: black;
	background: #d7dee3 url("tabla.gif") repeat-x top left;
}
td
{
	font-size: 100%;
	text-align: center;
	border: 1px dashed gray;
	color: black;
}
.Pie
{
	bottom: 0;
	left: 0;
	right: 0;
	height: 10%;
	position: fixed;
	background-color: #333;
	color: white;
	text-align: center;
}
</style>
