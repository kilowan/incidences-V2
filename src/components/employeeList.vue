<template>
  <div>
    <!-- employeeList -->
    <div id="employeList" v-if="mod=='employeeList'">
      <br /><table>
          <tr>
              <th>Lista de empleados</th>
          </tr>
      </table><br />
      <table>
          <tr>
              <th>DNI del empleado</th>
              <th>Nombre</th>
              <th>Primer apellido</th>
              <th>Segundo apellido</th>
              <th>Tipo de empleado</th>
              <th>--</th>
          </tr>
          <tr v-for="(employee, index) in employeesFiltered()" v-bind:key="index">
              <td>{{employee.dni}}</td>
              <td>{{employee.name}}</td>
              <td>{{employee.surname1}}</td>
              <td>{{employee.surname2}}</td>
              <td>{{employee.tipo}}</td>
              <td style="width:10%; height: 2%;">
                <b-link @click="deleteEmployee(employee.id)">
                  <img class="cierra" src="./delete.png" alt="Borrar empleado" style="width:12%; height: 12%;"/>
                </b-link>
                <b-link @click="edit(employee)">
                  <img class="cierra" src="./edit.png" alt="Editar empleado" style="width:12%; height: 12%;"/>
                </b-link>
                <b-link @click="panel(employee)">
                  <img class="cierra" src="./see.png" alt="Ver empleado" style="width:12%; height: 12%;"/>
                </b-link>
              </td>
          </tr>
          <tr>
            <td colspan="8">
                <b-link @click="$bvModal.show('new')">Agregar nuevo</b-link>
            </td>
          </tr>
      </table>
    </div>
    <div v-else-if="mod=='panel'" id="panel">
      <user-panel :user="employee" :incidences="incidences"/>
    </div>
    <b-modal id="warning" hide-header hide-footer>
      <div class="d-block text-center">
        <h3>¿Seguro que quieres borrar este empleado?</h3>
      </div>
      <div class="modal-footer">
        <b-button block @click="$bvModal.hide('warning')">Cancel</b-button>
        <b-button block @click="confirmDelete()">Ok</b-button>
      </div>
    </b-modal>
    <b-modal class="nuevoemp" id="new" hide-header hide-footer>
      <!--ok.prevent-->
      <div class="d-block text-center">
        <h1>Hoja del nuevo empleado:</h1><br />
        <input placeholder="DNI" v-model="dni"/><br />
        <input placeholder="Nombre" v-model="name"/><br />
        <input placeholder="Primer Apellido" v-model="surname1"/><br />
        <input placeholder="Segundo Apellido" v-model="surname2"/><br />
        <input placeholder="Username" v-model="username"/><br />
        <input placeholder="Contraseña" v-model="password"/><br />
        <b-form-select v-model="selected" :options="options" size="sm" class="mt-3"></b-form-select><br />
      </div>
      <div class="modal-footer">
        <b-button block @click="cancel('new')">Cancel</b-button>
        <b-button block @click="save()">Guardar</b-button>
      </div>
    </b-modal>
    <b-modal class="editemp" id="editemp" hide-header hide-footer>
      <div class="d-block text-center" v-if="user">
        <h1>Editar empleado</h1><br />
        <label>DNI:</label>
        <input disabled v-model="dni"/><br />
        <label>Nombre:</label>
        <input :disabled="tipo === 'Admin'" v-model="name"/><br />
        <label>Primer Apellido:</label>
        <input :disabled="tipo === 'Admin'" v-model="surname1"/><br />
        <label>Segundo Apellido:</label>
        <input :disabled="tipo === 'Admin'" v-model="surname2"/><br />
        <label>Tipo:</label>
        <b-form-select :disabled="tipo === 'Admin'" v-model="tipo" :options="options" size="sm" class="mt-3"></b-form-select>
      </div>
      <div class="modal-footer">
        <b-button block @click="cancel('editemp')">Cancel</b-button>
        <b-button :disabled="tipo === 'Admin'" block @click="update()">Actualizar</b-button>
      </div>
    </b-modal>
  </div>
</template>

<script>

import axios from 'axios';
import UserPanel from './userPanel.vue';

export default {
  name: 'employeeList',
  props: ['user', 'incidences'],
  components: {
    UserPanel,
  },
  data:function() {
    return {
      employees: undefined,
      employee: undefined,
      employeSelected: undefined,
      mod: 'employeeList',
      selectedToDelete: undefined,
      dni: undefined,
      name: undefined,
      surname1: undefined,
      surname2: undefined,
      username: undefined,
      password: undefined,
      tipo: undefined,
      fields: [],
      values: [],
      selected: null,
      options: [
        { value: null, text: 'Tipo', default: true},
        { value: 'Limpiador', text: 'Un limpiador' },
        { value: 'Encargado', text: 'Un encargado' },
        { value: 'Técnico', text: 'Un técnico' },
        { value: 'Becario', text: 'Un becario' },
        { value: 'Admin', text: 'Un administrador' },
        { value: 'Temporal', text: 'Uno temporal' },
        { value: 'Otro', text: 'Otro tipo aún no definido' }
      ]
    }
  },
  methods: {
    employeesFiltered(){
      return this.employees.filter(data => {
        return data.tipo !== 'Admin';
      })
    },
    update: function() {
      this.fillData([this.name, this.surname1, this.surname2, this.tipo]);
      if (this.fields.length >0) {
        axios({
          method: 'post',
          url: 'http://localhost:8082/newMenu.php',
          data: {
            funcion: 'updateWorker',
            dni: this.employeSelected.dni,
            fields: this.fields,
            values: this.values,
          },
          headers:[],
        }).then(() =>{
          this.cancel('editemp');
          this.load();
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
      this.pushField(data[0], this.employeSelected.name, "nombre");
      this.pushField(data[1], this.employeSelected.surname1, "apellido1");
      this.pushField(data[2], this.employeSelected.surname2, "apellido2");
      this.pushField(data[3], this.employeSelected.tipo, "tipo");
    },
    cancel: function(name) {
      if(name === 'new'){
        this.username = undefined;
        this.password = undefined;
      }
      this.tipo = undefined;
      this.dni = undefined;
      this.name = undefined;
      this.surname1 = undefined;
      this.surname2 = undefined;
      this.fields = [];
      this.values = [];
      this.$bvModal.hide(name);
    },
    save() {
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
          type: this.tipo,
        },
        headers:[],
      }).then(() =>{
          this.cancel('new')
          this.load();
        }
      );
    },
    panel: function(employee) {
      this.employee = employee;
      this.mod = 'panel';
    },
    edit: function(employee) {
      this.employeSelected = employee;
      this.name = employee.name;
      this.surname1 = employee.surname1;
      this.surname2 = employee.surname2;
      this.tipo = employee.tipo;
      this.dni = employee.dni;
      this.$nextTick(() => {
        this.$bvModal.show('editemp');
      });
    },
    add:function() {
      this.$nextTick(() => {
        this.$bvModal.show('new');
      });
    },
    reload: function() {
      this.load();
      this.mod = 'employeeList';
    },
    deleteEmployee: function(id) {
      this.selectedToDelete = id;
        this.$nextTick(() => {
          this.$bvModal.show('warning');
        });
    },
    confirmDelete: function() {
      axios({
      method: 'get',
      url: 'http://localhost:8082/employee.php?funcion=removeEmployee&id=' + this.selectedToDelete,
      })
      .then(()=> {
        this.$bvModal.hide('warning');
        this.load();
      });
    },
    load: function() {
      axios({
      method: 'get',
      url: 'http://localhost:8082/employee.php?funcion=getEmpolyeeList',
      })
      .then(data =>
        this.employees = data.data
      );
    },
  },
  mounted() {
    this.load();
  }
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
