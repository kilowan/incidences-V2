<template>
  <!--<div>-->
    <b-modal class="nuevoemp" id="new" hide-header hide-footer>
      <!--ok.prevent-->
      <div class="d-block text-center">
        <h1>Hoja del nuevo empleado:</h1><br />
        <label>DNI:</label>
        <input v-model="dni"/><br />
        <label>Nombre:</label>
        <input v-model="name"/><br />
        <label>Primer Apellido:</label>
        <input v-model="surname1"/><br />
        <label>Segundo Apellido:</label>
        <input v-model="surname2"/><br />
        <label>Username:</label>
        <input v-model="username"/><br />
        <label>Contraseña:</label>
        <input v-model="password"/><br />
        <label>Contraseña:</label>
        <input v-model="password"/><br />
        <p> ¿Que tipo de empleado es?:</p>
        <p>
            <select v-model="type" required>
                <option value="Limpiador">Un limpiador</option>
                <option value="Encargado">Un encargado</option>
                <option value="Tecnico">Un tecnico</option>
                <option value="Admin">Un administrador</option>
                <option value="Temporal">Uno temporal</option>
                <option value="Otro">Otro tipo aún no definido</option>
            </select>
        </p><br />
      </div>
      <div class="modal-footer">
        <b-button block @click="$bvModal.hide('new')">Cancel</b-button>
        <b-button block @click="save()">Guardar</b-button>
      </div>
    </b-modal>
    <!-- addEmployee -->
    <!--<div class="nuevoemp">
      <h1>Hoja del nuevo empleado:</h1><br />
      <label>DNI:</label>
      <input type="text" v-model="dni" required /><br />
      <label>Nombre:</label>
      <input type="text" v-model="name" required /><br />
      <label>Primer Apellido:</label>
      <input type="text" v-model="surname1" required /><br />
      <label>Segundo Apellido:</label>
      <input type="text" v-model="surname2" /><br />
      <label>Username:</label>
      <input type="username" v-model="username" required /><br />
      <label>Contraseña:</label>
      <input type="password" v-model="password" required /><br />
      <p> ¿Que tipo de empleado es?:</p>
      <p>
          <select v-model="type" required>
              <option value="Limpiador">Un limpiador</option>
              <option value="Encargado">Un encargado</option>
              <option value="Tecnico">Un tecnico</option>
              <option value="Admin">Un administrador</option>
              <option value="Temporal">Uno temporal</option>
              <option value="Otro">Otro tipo aún no definido</option>
          </select>
      </p><br />
      <a href="#" @click="save()">Guardar</a><br /> 
    </div><br />
    <div>
      <a href="#" @click="back()" class="link" center>Atrás</a>
    </div>-->
  <!--</div>-->
</template>

<script>

import axios from 'axios';

export default {
  name: 'addEmployee',
  props: ['user'],
  components: {
  },
  data:function()
  {
    return {
      username: undefined,
      password: undefined,
      name: undefined,
      surname1: undefined,
      surname2: undefined,
      type: undefined,
    }
  },
  methods: {
    save()
    {
      axios({
        method: 'post',
        url: 'http://localhost:8082/newMenu.php',
        data: {
          funcion: 'addEmployee',
          username: this.username,
          password: this.password,
          dni: this.dni,
          name: this.name,
          surname1: this.surname1,
          surname2: this.surname2,
          type: this.type,
        },
        headers:[],
      }).then(
        this.$emit('reload')
      );
    },
    back:function()
    {
      this.$emit('stepBack');
    }
  },
  mounted(){}
}
</script>
<style>
.nuevoemp
{
	text-align: center;
	border: 2px solid black;
  background-color: #d7dee3;
	left: 30%;
	width: 40%;
	position: relative;
}
</style>
