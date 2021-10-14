<template>
  <div v-if="user">
    <div class="cabecera">
      <p class="mensaje">Bienvenido {{user.name}} {{user.surname1}} {{user.surname2}}</p>
      <div class="Logo">
        <router-link @click="logOut()" to="/">
          <img class="cierra" src="../shutdown.png" alt="Cerrar sesión" />
        </router-link>
      </div>
      <div class="opciones">
        <b-link @click="$bvModal.show('make-incidence')" v-if="user.permissions.includes('13')" class="link">Crear parte</b-link>
        <b-link @click="add('incidences')" v-if="incidencesCount >0" class="link">Ver partes</b-link>
        <b-link @click="add('statistics')" v-if="user.permissions.includes('2')" class="link" >Estadísticas</b-link> 
        <b-link @click="add('employeeList')" v-if="user.permissions.includes('16')" class="link">Lista empleados</b-link>
        <b-link @click="add('user_info')" class="link" >Datos personales</b-link>
      </div>
    </div>
    <div class="cuerpo">
      <div v-if="check('user_info')">
        <user-info  
          v-if="user" 
          :username="username" 
          @reloadUser="reloadUser($event)
        "/>
      </div>
      <div v-else-if="check('statistics')">
        <statistics  v-if="user" :user="user"/>
      </div>
      <div v-else-if="check('employeeList')">
        <employee-list  
          v-if="user" 
          :user="user" 
          :incidences="incidences"
        />
      </div>
      <div v-else-if="check('incidences')">
        <incidences 
          v-if="user" 
          :user="user"
          :reload="reload"
          :incidences="incidences"
          @linked="reload=false"
          @reload="reloading()"
        />
      </div>
    </div>
    <div class="Pie">
      <p>Trabajo realizado por Jose Javier Valero Fuentes y Juan Francisco Navarro Ramiro para el curso de ASIR 2º X migrado a Vue.js</p>
    </div>
    <b-modal id="make-incidence" hide-header hide-footer>
      <div class="d-block text-center">
        <!-- MakeIncidence -->
        <h3>Crear parte</h3>
        <div v-if="pieces">
          <label>¿Que pieza/s crees que falla/n?:</label><br />
          <select v-model="selectedPiece" name="pieza">
            <option v-for="(piece, index) in pieces" v-bind:key="index">{{ piece.name }}</option>
          </select>
          <button v-if="selectedPiece" @click="addPiece()">Añadir</button>
          <button v-if="selectedPieces.length>0" @click="reset()">Reiniciar</button><br />
        </div>
        <input type="textarea" placeholder="Descripción del problema" v-model="description"/><br />
        <div v-if="selectedPieces.length>0">
          <label>Piezas seleccionas:</label>
          <p>{{ concatPieces() }}</p>
        </div>
      </div>
      <div class="modal-footer">
        <b-button block @click="cancel()">Cancel</b-button>
        <b-button :disabled="selectedPieces.length < 1 || !description" block @click="addIncidence()">Enviar</b-button>
      </div>
    </b-modal>
  </div>
</template>

<script>

import axios from 'axios';
import userInfo from './userInfo.vue';
import statistics from './statistics.vue';
import employeeList from './employeeList.vue';
import incidences from './incidences.vue';

export default {
  name: 'main',
  props: {
    /*message: {
      type: String,
      required: false
    },*/
  },
  components: {
    userInfo,
    statistics,
    employeeList,
    incidences,
  },
  data: function() {
    return {
      page: 'Login',
      mod: 'Main',
      user: undefined,
      incidences: undefined,
      incidencesCount: 0,
      reload: false,
      username: undefined,
      selected: undefined,
      pieces: [],
      checked: false,
      choosen: '--',
      description: undefined,
      selectedPiece: undefined,
      selectedPieces: [],
      PieceIdsSelected: [],
    }
  },
 methods: {
   cancel: function() {
     this.$bvModal.hide('make-incidence');
      this.selected = undefined;
      this.checked = false;
      this.choosen = '--';
      this.description = undefined;
      this.selectedPiece = undefined;
      this.selectedPieces = [];
      this.PieceIdsSelected = [];
   },
    addPiece: function() {
      if (!this.selectedPieces.includes(this.selectedPiece)) {
        if (this.selectedPieces.includes('no sé') || this.selectedPiece == 'no sé') {
          this.reset();
        }
        this.selectedPieces.push(this.selectedPiece);
        let piece = this.pieces.filter(data =>{
          return data.name == this.selectedPiece;
        })[0];
        this.PieceIdsSelected.push(piece.id);
      }
    },
    reset: function() {
      this.selectedPieces = []
    },
    addIncidence: function() {
      axios({
        method: 'post',
        url: 'http://localhost:8082/newMenu.php',
        data: {
          funcion: 'addIncidence',
          ownerId: this.user.id,
          issueDesc: this.description,
          pieces: this.PieceIdsSelected,
        },
      headers:[]
      })
      .then(
        this.$emit('closeForm')
      );
    },
    checkForm: function() {
      return this.selected && this.description? true:false;
    },
    getPiece: function() {
      return this.pieces.filter(piece => {
        return piece.name == this.selected;
      })[0];
    },
    concatPieces: function() {
      return this.selectedPieces.join(', ');
    },
    check: function(data) {
      return this.mod == data? true: false;
    },
    reloading: function() {
        axios.get("http://localhost:8082/newMenu.php?funcion=getAllincidences")
        .then( datas => {
          this.incidences = datas.data;
          this.reload = true;
        });
    },
    logedIn: function(data) {
      axios.get("http://localhost:8082/employee.php?funcion=getEmployeeByUsername&username="+ data)
      .then( datas => {
        this.user = datas.data;
        this.username = data;
        this.page = 'Menu';
        axios.get("http://localhost:8082/newMenu.php?funcion=getAllincidences")
        .then( datas => {
          this.incidences = datas.data;
          this.showIncidences();
        });
      });
    },
    reloadUser: function(data) {
      axios.get("http://localhost:8082/employee.php?funcion=getEmployeeByUsername&username="+ data)
      .then( datas => {
        this.user = datas.data;
      });
    },
    add: function(data) {
      if (data == 'incidences' && this.mod =='incidences') {
        this.reloading();
      }
      this.mod = data;
    },

    showIncidences: function() {
      let new_array = undefined;
      if(this.user.permissions.includes("7") && this.user.permissions.includes("8") && this.user.permissions.includes("9"))
      {
          new_array = this.incidences.filter(array => {
              return (array.owner.id == this.user.id && array.state != 5);
          });
          this.incidencesCount = new_array.length;
      } else if (this.user.permissions.includes("3") && this.user.permissions.includes("4") && this.user.permissions.includes("5")) {
          new_array = this.incidences.filter(array => {
              return (array.solver.id == this.user.id || array.state == 1);
          });
          this.incidencesCount = new_array.length;
      } else if (this.user.permissions.includes("6") && this.user.permissions.includes("7") && this.user.permissions.includes("8") && this.user.permissions.includes("9") && this.user.permissions.includes("10") && this.user.permissions.includes("11") && this.user.permissions.includes("12")) {
          new_array = this.incidences.filter(array => {
              return (array.solver.id == this.user.id || (array.state == 1 || array.state == 2 || array.state == 3 || array.state == 4) || array.owner.id == this.user.id);
          });
          this.incidencesCount = new_array.length;
          this.page = 'Menu';
      }
    },
    logOut: function() {
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
  mounted() {
    if(this.$route.params.username) this.logedIn(this.$route.params.username);
    axios.get("http://localhost:8082/newMenu.php?funcion=getPiecesList")
      .then( data => {
        this.pieces = data.data;
    });
  }
}
</script>
<style>
  .cabecera {
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
  .crearP {
    text-align: center;
    border: 2px solid black;
    background-color: #d7dee3;
    left: 30%;
    width: 40%;
    position: absolute;
  }
</style>
