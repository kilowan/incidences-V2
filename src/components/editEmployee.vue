<template>
<!-- editEmployee -->
  <div>
      <br />
    <table>
      <tr>
          <th>Editar empleado</th>
      </tr>
    </table><br />
    <table>
        <tr>
            <th>DNI</th>
            <th>Nombre</th>
            <th>Primer apellido</th>
            <th>Segundo apellido</th>
            <th>Tipo</th>
            <th>--</th>
        </tr>
        <tr v-if="user">
            <td><input type="text" disabled name="nombre" v-model="dni" required /></td>
            <td><input type="text" name="nombre" v-model="name" required /></td>
            <td><input type="text" name="apellido1" v-model="surname1" required /></td>
            <td><input type="text" name="apellido2" v-model="surname2" /></td>
            <td><input type="text" name="tipo" v-model="tipo" required /></td>
            <td><a href="#" @click="save()">Guardar</a></td>
        </tr>
    </table><br/>
    <a href="#" @click="back()" class="link" center>Atrás</a>
  </div>
    <!--<b-modal class="editemp" id="editemp" hide-header hide-footer>
      <div class="d-block text-center" v-if="user">
        <h1>Editar empleado</h1><br />
        <label>DNI:</label>
        <input disabled v-model="dni"/><br />
        <label>Nombre:</label>
        <input v-model="name"/><br />
        <label>Primer Apellido:</label>
        <input v-model="surname1"/><br />
        <label>Segundo Apellido:</label>
        <input v-model="surname2"/><br />
        <label>Tipo:</label>
        <input v-model="tipo" required />
      </div>
      <div class="modal-footer">
        <b-button block @click="$bvModal.hide('editemp')">Cancel</b-button>
        <b-button block @click="save()">Guardar</b-button>
      </div>
    </b-modal>-->
    <!--ok.prevent-->
</template>

<script>

import axios from 'axios';

export default {
  name: 'editEmployee',
  props: ['user'],
  components: {
  },
  data:function() {
    return {
      dni: undefined,
      name: undefined,
      surname1: undefined,
      surname2: undefined,
      tipo: undefined,
      fields: [],
      values: [],
    }
  },
  methods: {
    save() {
      this.fillData([this.name, this.surname1, this.surname2, this.tipo]);
      if (this.fields.length >0) {
        axios({
          method: 'post',
          url: 'http://localhost:8082/newMenu.php',
          data: {
            funcion: 'updateWorker',
            dni: this.user.dni,
            fields: this.fields,
            values: this.values,
          },
          headers:[],
        }).then(() =>{
          this.$emit('reload');
        });
      }
    },
    reset: function() {
      this.name = undefined;
      this.surname1 = undefined;
      this.surname2 = undefined;
      this.tipo = undefined;
      this.dni = undefined;
      this.fields = [];
      this.values = [];
    },
    back:function() {
      this.$emit('stepBack');
    },
    checkField(field, field2) {
      return field && field != field2? true: false
    },
    pushField(data, parity, name) {
      if(this.checkField(data, parity))
      {
        this.values.push(data);
        this.fields.push(name);
      }
    },
    fillData(data) {
      this.pushField(data[0], this.user.name, "nombre");
      this.pushField(data[1], this.user.surname1, "apellido1");
      this.pushField(data[2], this.user.surname2, "apellido2");
      this.pushField(data[3], this.user.tipo, "tipo");
    },
  },
  mounted() {
      this.name = this.user.name;
      this.surname1 = this.user.surname1;
      this.surname2 = this.user.surname2;
      this.tipo = this.user.tipo;
      this.dni = this.user.dni;
  }
}
</script>
<style></style>
