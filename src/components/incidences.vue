<template>
    <!-- own incidences -->
    <br /><div v-if="!incidence">
      <table>
        <row 
          v-if="infoData"
          :data="infoData"
          :header="true"
          @select-tab="selectTab($event)"
        />
      </table><br />
      <div v-if="checkPermissions(user.permissions, ['6', '7', '8', '9'])">
        <!-- new -->
        <incidences-view v-if="newOwnIncidences && tab=='new'"
        :incidences="newOwnIncidences"
        :user="user"
        :admin="admin"
        :title="'Partes nuevos propios'"
        @linked="linked($event)"/>

        <!-- attended -->
        <incidences-view v-if="attendedOwnIncidences && tab=='current'"
        :incidences="attendedOwnIncidences"
        :user="user"
        :admin="admin"
        :title="'Partes atendidos propios'"
        @linked="linked($event)"/>

        <!-- closed -->
        <incidences-view v-if="closedOwnIncidences && tab=='old'"
        :incidences="closedOwnIncidences"
        :user="user"
        :admin="admin"
        :title="'Partes cerrados propios'"
        @linked="linked($event)"/>

        <!-- hidden -->
        <incidences-view v-if="hiddenOwnIncidences && tab=='hidden'"
        :incidences="hiddenOwnIncidences"
        :user="user"
        :admin="admin"
        :title="'Partes ocultos propios'"
        @linked="linked($event)"/>
      </div>
      <!-- other incidences -->
      <div v-if="checkPermissions(user.permissions, ['10', '11', '12']) || checkPermissions(user.permissions, ['3', '4', '5'])">
        <!-- new -->
        <incidences-view v-if="newIncidences && tab=='new'"
        :incidences="newIncidences"
        :user="user"
        :admin="admin"
        :title="'Partes nuevos'"
        @linked="linked($event)"
        @reload="reloading()"/>

        <!-- attended -->
        <incidences-view v-if="attendedIncidences && tab=='current'"
        :incidences="attendedIncidences"
        :user="user"
        :admin="admin"
        :title="'Partes atendidos'"
        @linked="linked($event)"
        @reload="reloading()"/>

        <!-- closed -->
        <incidences-view v-if="closedIncidences && tab=='old'"
        :incidences="closedIncidences"
        :user="user"
        :admin="admin"
        :title="'Partes cerrados'"
        @linked="linked($event)"
        @reload="reloading()"/>
      </div>
    </div>
    <div v-else>
      <incidence-view 
        v-if="incidence && checkreload()"
        :user="user" 
        :incidence="incidence"
        @reload="reloading()"
        @stepBack="back()"
        />
    </div>
</template>

<script>

import incidencesView from './incidencesView.vue';
import incidenceView from './incidenceView.vue';
import row from '../custom/row.vue';

export default {
  name: 'incidences',
  props: ['user', 'incidences', 'admin', 'reload'],
  components: {
    incidencesView,
    incidenceView,
    row,
  },
  data:function()
  {
    return {
      newOwnIncidences: [],
      attendedOwnIncidences: [],
      closedOwnIncidences: [],
      hiddenOwnIncidences: [],
      closedIncidences: [],
      attendedIncidences: [],
      newIncidences: [],
      incidence: undefined,
      tab: 'new',
      infoData: undefined,
    }
  },
  methods: {
    linked:function(incidence)
    {
      this.incidence = incidence;
      this.$emit('linked');
    },
    checkPermissions: function(permissions, permissionNumbers)
    {
      let result = true;
      permissionNumbers.forEach(element => {
        if (!permissions.includes(element)) {
          result = false;
        }
      });
      return result;
    },
    reloading: function()
    {
      this.incidence = undefined;
      this.tab = 'new';
      this.$emit('reload');
    },
    checkreload: function()
    {
      if (!this.reload) {
        return true
      } else {
        this.incidence = undefined;
        return false;
      }
    },
    selectTab:function(tab)
    {
      this.tab = tab;
    },
    back: function()
    {
      this.incidence = undefined;
    },
    handle: function()
    {
      this.incidence = undefined;
      if (this.checkPermissions(this.user.permissions, ['6', '7', '8', '9'])) {
        this.newOwnIncidences = this.incidences.filter(data => {
          return data.owner.dni == this.user.dni && data.state == 1;
        });
        this.attendedOwnIncidences = this.incidences.filter(data => {
          return data.owner.dni == this.user.dni && data.state == 2;
        });
        this.closedOwnIncidences = this.incidences.filter(data => {
          return data.owner.dni == this.user.dni && data.state == 3;
        });
        this.hiddenOwnIncidences = this.incidences.filter(data => {
          return data.owner.dni == this.user.dni && data.state == 4;
        });
      }
      if (this.checkPermissions(this.user.permissions, ['10', '11', '12']) || this.checkPermissions(this.user.permissions, ['3', '4', '5'])) {
        this.newIncidences = this.incidences.filter(data => {
          return data.state == 1 && data.owner.id != this.user.id;
        });
        this.attendedIncidences = this.incidences.filter(data => {
          return data.solver.dni == this.user.dni && data.state == 2;
        });
        this.closedIncidences = this.incidences.filter(data => {
          return data.solver.dni == this.user.dni && data.state == 3;
        });
      }
      this.infoData = [
        {
          hasData: this.newOwnIncidences.length>0 || this.newIncidences.length>0? true: false,
          inName: 'Nuevos',
          outName: 'new',
        },
        {
          hasData: this.attendedOwnIncidences.length>0 || this.attendedIncidences.length>0? true: false,
          inName: 'Atendidos',
          outName: 'current',
        },
        {
          hasData: this.closedOwnIncidences.length>0 || this.closedIncidences.length>0? true: false,
          inName: 'Cerrados',
          outName: 'old',
        },
      ];
      if (this.hiddenOwnIncidences.length>0) {
        this.infoData.push({
          hasData: this.hiddenOwnIncidences.length>0? true: false,
          inName: 'Ocultos',
          outName: 'hidden',
        });
      }
    },
  },
  mounted(){
    this.handle();
  }
}
</script>
<style></style>
