<template>
  <div class="hello" style="height: 100px">
    <table border="1">
      <tbody>
        <tr>
          <td v-for="block, key of blocks" :key="key" :style="`background-color:${block.color};`">
          {{block.status}}
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
// const STATUS = {
//   FREE:0,
//   ALLOCATED:1,
//   TRANSPARENT: 2,
//   ALLOCATED_AND_OVERWRITTEN: 3,
//   OVERWRITTEN: 4,
// };

export default {
  name: 'FileSystem',
  props: {
    blocks: Array
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
