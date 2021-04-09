<template>
  <div class="hello" style="height: 100px">
    <table border="1">
      <tbody>
        <tr>
          <td v-for="block, key of blocks" :key="key" :style="`background-color:${block.color};`">
          {{typeof block.fid === 'number'?block.fid:''}}
          </td>
        </tr>
      </tbody>
      <tbody>
        <tr>
          <td v-for="block, key of blocks" :key="key" :style="`background-color:${colorMap(block.allocateCounter/50)};`">
          {{block.allocateCounter}}
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
import * as d3 from "d3"
export default {
  name: 'FileSystem',
  props: {
    blocks: Array
  },
  data(){
    return {
      colorMap : d3.interpolateRgb(
        d3.rgb('#ebedf0'),
        d3.rgb('#216e39')
      )
    }
  }
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
td {
  width:2rem;
  height: 2rem;
}
a {
  color: #42b983;
}
</style>
