<template>
  <lx-empty>
    <th v-if="header && html" v-html="html" />
    <th v-else-if="header && link" >
      <a href="#" @click="$emit('selectTab')">{{ text }}</a>
    </th>
    <th v-else-if="header">{{ text }}</th>
    <td v-else>{{ text }}</td>
  </lx-empty>
</template>

<script>

export default {
  name: 'cell',
  props: ['header', 'text', 'link'],
  components: {
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
