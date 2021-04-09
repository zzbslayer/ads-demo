<template>
  <div class="hello">
    <file-system v-for="blocks, key of fileSystems" :key="key" :blocks="blocks"/>
  </div>
</template>

<script>
import FileSystem from './FileSystem.vue';

const STATUS = {
  FREE:0,
  ALLOCATED:1,
  TRANSPARENT: 2,
  ALLOCATED_AND_OVERWRITTEN: 3,
  OVERWRITTEN: 4,
};

const COLOR = {
  0: '#ffffff',
  1: '#1a75ff',
  2: '#66ff66',
  3: '#ff66ff',
  4: '#ff3333',
}

let fid = 0;

export default {
  components: { FileSystem },
  name: 'HelloWorld',
  props: {
    msg: String
  },
  data: function() {
    const number = 3;
    const fileSystems = [];
    for (const i of Array(number).keys()) {
      fileSystems[i]=(this.generateBlocks(50));
    }

    return {
      number,
      fileSystems,
      files: [],
    }
  },
  mounted() {
    this.setRandomOpsWithTimeout();
  },

  methods: {
    print(a) { console.log(a) },
    setRandomOpsWithTimeout() {
      this.lastTimeout = setTimeout(() => {
        this.randomOps();
        this.setRandomOpsWithTimeout();
      }, 1000);
      
    },
    randomInt(max) {
      return Math.floor(Math.random() * max);
    },
    randomOps() {
      //const opsList = [this.writeOrdinary, this.readOrdinary, this.writeTransparent, this.readTransparent];
      const opsList = [this.writeOrdinary, this.readOrdinary, this.writeTransparentAvoidHotArea, this.readTransparent, this.deleteOrdinary];
      opsList[this.randomInt(opsList.length)]();
    },
    getRandomBlocks() {
      const index = this.randomInt(this.fileSystems.length);
      return { blocks: this.fileSystems[index], index };
    },
    getTwoRandomBlocks() {
      const index1 = this.randomInt(this.fileSystems.length);
      const index2 = (index1 + 1) % this.fileSystems.length;
      return [
        { blocks: this.fileSystems[index1], index: index1},
        { blocks: this.fileSystems[index2], index: index2}
      ];
    },
    deleteOrdinary(){
      console.warn("delete ordinary");
      let shuffleFiles = Array.from(this.files).sort(function() {
        return .5 - Math.random();
      });

      const _clearBlock = (block) => {
        block.status = STATUS.FREE;
        block.color = COLOR[block.status]
        block.fid = null;
        let p = block.nextBlock;
        block.nextBlock = null;
        return p;
        // block.allocateCounter +=1;
      }

      for(let file of shuffleFiles){
        if(file.type === 'ordinary'){
          this.files = this.files.filter((f)=>f.fid!==file.fid)
          let location = file.location[0];
          let blockHead = this.fileSystems[location.fsIndex][location.blockIndex]
          while(blockHead){
            blockHead = _clearBlock(blockHead)
          }
          break;
        }
      }
    },
    writeOrdinary() {
      console.warn("write ordinary")
      const res = this.getRandomBlocks();
      const randomBlocks = res.blocks;
      const fsIndex = res.index;

      const currentFid = fid++;

      const _canWrite = (block) => {
        if (!block)
          return false;
        if (block.status === STATUS.FREE || block.status === STATUS.TRANSPARENT || block.status === STATUS.OVERWRITTEN)
          return true;
        return false;
      };

      const _writeBlock = (block) => {
        if (block.status === STATUS.FREE) {
          block.status = STATUS.ALLOCATED;
        }
        else if (block.status === STATUS.TRANSPARENT || block.status === STATUS.OVERWRITTEN) {
          block.status = STATUS.ALLOCATED_AND_OVERWRITTEN;
        }
        block.color = COLOR[block.status]
        block.fid = currentFid;
        block.allocateCounter +=1;
      }
      
      // file size is 2 blocks
      for (const i in randomBlocks) {
        const block = randomBlocks[i];
        const nextBlock = randomBlocks[Number(i)+1];
        if (_canWrite(block) && _canWrite(nextBlock)) {
          _writeBlock(block);
          block.nextBlock = nextBlock;
          _writeBlock(nextBlock);
          nextBlock.nextBlock = null;
          this.files.push({ fid: currentFid, type: "ordinary",location:[{ fsIndex, blockIndex: i, size: 2}] })
          break;
        }
      }
      console.log(this.files)
      
    },
    readOrdinary() {
      console.warn("read ordinary")
    },
    writeTransparent() {
      console.warn("write transparent")
      const res = this.getTwoRandomBlocks();

      const currentFid = fid++;

      const _canWrite = (block) => {
        if (!block)
          return false;
        if (block.status === STATUS.FREE)
          return true;
        return false;
      };

      const _writeBlock = (block) => {
        if (block.status === STATUS.FREE) {
          block.status = STATUS.TRANSPARENT;
        }
        block.color = COLOR[block.status]
        block.fid = currentFid;
      }

      const file = { fid: currentFid, type: "transparent", location: [] }
      
      // file size is 2 blocks
      const first = res[0].blocks
      for (const i in first) {
        const block = first[i];
        const nextBlock = first[Number(i)+1];
        if (_canWrite(block) && _canWrite(nextBlock)) {
          _writeBlock(block);
          _writeBlock(nextBlock);
          file.location.push({ fsIndex: res[0].index, blockIndex: i, size: 2});
          break;
        }
      }

      const second = res[1].blocks
      for (const i in second) {
        const block = second[i];
        const nextBlock = second[Number(i)+1];
        if (_canWrite(block) && _canWrite(nextBlock)) {
          _writeBlock(block);
          _writeBlock(nextBlock);
          file.location.push({ fsIndex: res[1].index, blockIndex: i, size: 2});
          break;
        }
      }
      this.files.push(file);
      console.log(this.files)
    },
    
    writeTransparentAvoidHotArea() {
      console.warn("write transparent")
      const res = this.getTwoRandomBlocks();

      const currentFid = fid++;

      const _canWrite = (block) => {
        if (!block)
          return false;
        if (block.status === STATUS.FREE)
          return true;
        return false;
      };

      const _writeBlock = (block) => {
        if (block.status === STATUS.FREE) {
          block.status = STATUS.TRANSPARENT;
        }
        block.color = COLOR[block.status]
        block.fid = currentFid;
      }

      const file = { fid: currentFid, type: "transparent", location: [] }
      
      // file size is 2 blocks
      const first = res[0].blocks
      for(let i = first.length-1; i >= 0; i--){
        const block = first[i];
        const nextBlock = first[Number(i)+1];
        if (_canWrite(block) && _canWrite(nextBlock)) {
          _writeBlock(block);
          _writeBlock(nextBlock);
          file.location.push({ fsIndex: res[0].index, blockIndex: i, size: 2});
          break;
        }
      }

      const second = res[1].blocks
      for(let i = second.length-1; i >= 0; i--){
        const block = second[i];
        const nextBlock = second[Number(i)+1];
        if (_canWrite(block) && _canWrite(nextBlock)) {
          _writeBlock(block);
          _writeBlock(nextBlock);
          file.location.push({ fsIndex: res[1].index, blockIndex: i, size: 2});
          break;
        }
      }
      this.files.push(file);
      console.log(this.files)
    },
    readTransparent() {
      console.warn("read transparent")
    },
    generateBlocks(number) {
      const blocks = [];
      for (const i of Array(number).keys()) {
        const block = {id: i, status: STATUS.FREE, color: COLOR[STATUS.FREE], allocateCounter: 0, nextBlock: null};
        blocks.push(block);
      }
      return blocks;
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
a {
  color: #42b983;
}
</style>
