<template>
  <div id="app">
    <router-view/>
  </div>
</template>
<script>
import axios from 'axios';
export default {
  name: 'Login',
  //components: {home},
  data:function()
  {
    return {
      form: {
        username: undefined,
        pass: undefined,
      },
      user: undefined,
      incidences: undefined,
    }
  },
  methods: {
    onSubmit: function()
    {
      axios.get("http://localhost:8082/newMenu.php?funcion=checkCredentials&username="+ this.form.username+"&pass="+this.form.pass)
      .then( data => {
        this.$router.push(
			{
				name: 'main', params: {
					username: data.data.username
					}
				}
			);
      });
    },
  },
  mounted(){}
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

#nav a.router-link-exact-active {
  color: #42b983;
}
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
