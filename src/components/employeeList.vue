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
          <tr v-for="(employee, index) in employees" v-bind:key="index">
              <td>{{employee.dni}}</td>
              <td>{{employee.name}}</td>
              <td>{{employee.surname1}}</td>
              <td>{{employee.surname2}}</td>
              <td>{{employee.tipo}}</td>
              <td style="width:10%; height: 2%;">
                <a @click="deleteEmployee(employee.id)" href="#">
                  <img class="cierra" src="./delete.png" alt="Borrar empleado" style="width:12%; height: 12%;"/>
                </a>
                <a @click="edit(employee)" href="#">
                  <img class="cierra" src="./edit.png" alt="Editar empleado" style="width:12%; height: 12%;"/>
                </a>
                <a @click="panel(employee)" href="#">
                  <img class="cierra" src="./see.png" alt="Ver empleado" style="width:12%; height: 12%;"/>
                </a>
              </td>
          </tr>
          <tr>
            <td colspan="8">
                <a href="#" @click="$bvModal.show('new')">Agregar nuevo</a>
            </td>
          </tr>
      </table>
    </div>
    <div v-else-if="mod=='panel'" id="panel">
      <user-panel :user="user" :incidences="incidences"/>
    </div>
    <div v-else-if="mod=='edit'" id="edit">
      <edit-employee 
      :user="employeSelected"
      @stepBack="mod = 'employeeList'"
      @reload="reload()"/>
    </div>
      <add-employee 
        @stepBack="mod = 'employeeList'"
        @reload="reload()"
      />
      <b-modal id="warning" hide-header hide-footer>
        <div class="d-block text-center">
          <h3>¿Seguro que quieres borrar este empleado?</h3>
        </div>
        <div class="modal-footer">
          <b-button block @click="$bvModal.hide('warning')">Cancel</b-button>
          <b-button block @click="confirmDelete()">Ok</b-button>
        </div>
      </b-modal>
  </div>
</template>

<script>

import axios from 'axios';
import UserPanel from './userPanel.vue';
import editEmployee from './editEmployee.vue';
import addEmployee from './addEmployee.vue';

export default {
  name: 'userInfo',
  props: ['user', 'incidences'],
  components: {
    UserPanel,
    editEmployee,
    addEmployee,
  },
  data:function()
  {
    return {
      employees: undefined,
      employee: undefined,
      employeSelected: undefined,
      mod: 'employeeList',
      selectedToDelete:undefined
    }
  },
  methods: {
    panel: function(employee)
    {
      this.employee = employee;
      this.mod = 'panel';
    },
    edit: function(employee)
    {
      this.employeSelected = employee;
      this.mod = 'edit';
    },
    add:function()
    {
      this.mod = 'add';
    },
    reload: function()
    {
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
    load: function()
    {
      axios({
      method: 'get',
      url: 'http://localhost:8082/employee.php?funcion=getEmpolyeeList',
      })
      .then(data =>
        this.employees = data.data
      );
    },
  },
  mounted(){
    this.load();
  }
}
</script>
<style></style>
