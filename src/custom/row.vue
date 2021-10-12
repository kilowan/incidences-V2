<template v-if="data">
  <tr>
    <cell
      v-for="(info, index) in data" v-bind:key="index"
      :header="header" 
      :link="info.hasData" 
      :text="info.inName" 
      @selectTab="$emit('selectTab', info.outName)"
  />
  </tr>
</template>

<script>

import cell from '../custom/cell.vue';

export default {
  name: 'row',
  props: ['header', 'data'],
  components: {
    cell
  },
  data:function()
  {
    return {}
  },
  methods: {},
  mounted(){}
}
</script>
<style></style>
