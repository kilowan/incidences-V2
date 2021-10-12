<template>
  <div>
    <!-- MakeIncidence -->
    <br /><table>
      <tr>
        <th>Crear parte</th>
      </tr>
    </table>
    <table>
      <tr>
        <td>Descripción del problema:</td>
        <td><input type="textarea" placeholder="resumen del fallo" v-model="description" rows="10" cols="40"/></td>
      </tr>
    </table><br />
    <table>
        <tr v-if="pieces">
          <th>¿Que pieza/s crees que falla/n?:</th>
        </tr>
    </table>
    <table>
        <tr>
          <td>
              <select v-model="selectedPiece" name="pieza">
                <option v-for="(piece, index) in pieces" v-bind:key="index">{{ piece.name }}</option>
              </select>
            <button v-if="selectedPiece" @click="addPiece()">Añadir</button>
            <button v-if="selectedPieces.length>0" @click="reset()">Reiniciar</button>
          </td>
        </tr>
    </table><br />
    <div v-if="selectedPieces.length>0">
      <table>
        <tr>
          <th>Piezas seleccionas:</th>
        </tr>
      </table>
      <table>
        <tr v-for="(selectedPiece, index) in selectedPieces" v-bind:key="index">
          <td v-text="selectedPiece"/>
        </tr>
      </table><br />
    </div>
    <a href="#" v-if="selectedPieces.length>0 && description" @click="addIncidence()" class="link" center>Enviar</a>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'makeincidence',
  props: ['user'],
  components: {
  },
  data:function()
  {
    return {
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
    addPiece: function()
    {
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
    reset: function()
    {
      this.selectedPieces = []
    },
    addIncidence: function()
    {
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
    checkForm: function()
    {
      return this.selected && this.description? true:false;
    },
    getPiece:function()
    {
      return this.pieces.filter(piece => {
        return piece.name == this.selected;
      })[0];
    },
  },
  mounted(){
    axios.get("http://localhost:8082/newMenu.php?funcion=getPiecesList")
      .then( data => {
        this.pieces = data.data;
    });
  }
}
</script>
<style>
.crearP
{
	text-align: center;
	border: 2px solid black;
  background-color: #d7dee3;
	left: 30%;
	width: 40%;
	position: absolute;
}
</style>
