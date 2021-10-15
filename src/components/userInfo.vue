<template>
  <div v-if="user">
    <!-- userInfo -->
    <br />
      <b-container :style="style">
        <b-row><b-col>Datos personales</b-col></b-row>
        <b-row>
          <b-col><label>DNI: </label></b-col>
          <b-col>{{ user.dni }}</b-col>
        </b-row>
        <b-row v-if="!edit">
          <b-col><label >Nombre: </label></b-col>
          <b-col>{{ user.name }}</b-col>
        </b-row>
        <b-row v-if="edit">
          <b-col><label>Nombre: </label></b-col>
          <b-col><input type="text" v-model="name"/></b-col>
        </b-row>
        <b-row v-if="!edit">
          <b-col><label >Primer apellido: </label></b-col>
          <b-col>{{ user.surname1 }}</b-col>
        </b-row>
        <b-row v-if="edit">
          <b-col><label >Primer apellido:</label>/</b-col>
          <b-col><input type="text" v-model="surname1"/></b-col>
        </b-row>
        <b-row v-if="!edit">
          <b-col><label >Segundo apellido: </label></b-col>
          <b-col>{{ user.surname2 }}</b-col>
        </b-row>
        <b-row v-if="edit">
          <b-col><label>Segundo apellido: </label></b-col>
          <b-col><input type="text" v-model="surname2"/></b-col>
        </b-row>
        <b-row>
          <b-col><label>Tipo: </label></b-col>
          <b-col>
            <b-form-select :disabled="true" v-model="tipo" :options="options" size="sm" class="mt-3"></b-form-select>
          </b-col>
        </b-row>
        <b-row v-if="edit" colspan="2">
          <b-col><a href="#" @click="saveData()">Guardar</a> <a href="#" @click="reset()">Reiniciar</a></b-col>
        </b-row>
        <b-row v-if="!edit">
          <b-col><a href="#" @click="editData()">Editar</a></b-col>
        </b-row>
    </b-container><br />
  </div>
</template>

<script>

import axios from 'axios';

export default {
  name: 'userInfo',
  props: ['username', 'userData'],
  components: {
  },
  data:function()
  {
    return {
      user: undefined,
      edit: false,
      name: undefined,
      surname1: undefined,
      surname2: undefined,
      fields: [],
      values: [],
      style: {
        //boxShadow: '5px 5px 10px #999',
        border: '1px dotted black',
        background: 'white',
        //left: '10%',
        width: '80%',
        position: 'relative',
        //borderSpacing: '0px'
      }
    }
  },
  methods: {
    pushField(data, parity, name)
    {
      if(this.checkField(data, parity))
      {
        this.values.push(data);
        this.fields.push(name);
      }
    },
    editData: function() {
      this.edit = true;
      this.name = this.user.name;
      this.surname1 = this.user.surname1;
      this.surname2 = this.user.surname2;
    },
    checkField(field, field2)
    {
      return field && field != field2? true: false
    },
    fillData(data)
    {
      this.pushField(data[0], this.user.name, "nombre");
      this.pushField(data[1], this.user.surname1, "apellido1");
      this.pushField(data[2], this.user.surname2, "apellido2");
    },
    saveData: function()
    {
      this.fillData([this.name, this.surname1, this.surname2]);
      if (this.fields.length >0) 
      {
        axios({
          method: 'post',
          url: 'http://localhost:8082/newMenu.php',
          data: {
            funcion: 'updateWorker',
            dni: this.user.dni? this.user.dni: this.userData.dni,
            fields: this.fields,
            values: this.values,
          },
          headers:[],
        }).then(
          this.$emit('reload')
        );        
      }
      this.$emit('reloadUser', this.user.dni);
      this.reset();
      if (!this.userData) {
        this.reloadUser();
      }
    },
    reset: function()
    {
      this.edit = false;
      this.name = undefined;
      this.surname1 = undefined;
      this.surname2 = undefined;
      this.fields = [];
      this.values = [];
    },
    reloadUser: function()
    {
      axios.get("http://localhost:8082/newMenu.php?funcion=getEmployeeByUsername&username="+ this.username)
      .then( datas => {
        this.user = datas.data;
      });
    },
  },
  mounted(){
    if (!this.userData) {
      axios.get("http://localhost:8082/newMenu.php?funcion=getEmployeeByUsername&username="+ this.username)
      .then( datas => {
        this.user = datas.data;
      });
    } else {
      this.user = this.userData;
    }
  }
}
</script>
<style></style>
