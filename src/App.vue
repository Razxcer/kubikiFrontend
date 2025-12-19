<script setup>
import { ref } from 'vue';
import { defineProps, watch } from 'vue';
import RowTiles from './components/RowTiles.vue';
import WorkTiles from './components/WorkTiles.vue';

const score = ref(0)
const gameSize = 600
const tileSize = gameSize*0.125
const offsetpixel = 10
const linesDoneX = ref([])
const linesDoneY = ref([])
const matrix = ref([
  [0,0,0,0,0,0,0,0],
  [0,0,0,0,0,0,0,0],
  [0,0,0,0,0,0,0,0],
  [0,0,0,0,0,0,0,0],
  [0,0,0,0,0,0,0,0],
  [0,0,0,0,0,0,0,0],
  [0,0,0,0,0,0,0,0],
  [0,0,0,0,0,0,0,0]
])  

const matrixParts = ref([
  [
  [1,1],
  [1,1]
  ],

  [
  [1,1,1],
  [0,0,1]
  ],

  [
  [1,1,1],
  [1,1,1],
  [1,1,1]
  ],

  [
  [1,1,1],
  [0,1,0]
  ],

  [
  [1,1,1],
  [1,1,1]
  ],

  [
  [0,0,1],
  [1,1,1]
  ],

  [
  [1,1,1]
  ],

  [
  [1,1,1,1]
  ],

  [
  [1],
  [1],
  [1],
  [1]
  ],

  [
  [1],
  [1],
  [1]
  ]
])


const partMatrix = ref([
  [1,1],
  [1,1]
])

watch(matrix.value,()=>{

  for(let i=0;i<matrix.value.length;i+=1)
  {
    let flagVert = true
    let flagHoriz = true
    for(let j=0;j<matrix.value[i].length;j+=1)
    {
      if(matrix.value[i][j] < 1) flagHoriz = false
    }

    for(let j=0;j<matrix.value.length;j+=1)
    {
      if(matrix.value[j][i] < 1) flagVert = false
    }

    if(flagHoriz){
      linesDoneX.value.push(i);
    }
    if(flagVert){
      linesDoneY.value.push(i);
    }
  }

  for(let i=0;i<linesDoneX.value.length;i+=1)
  {
    for(let j=0;j<matrix.value[0].length;j+=1)
    {
      matrix.value[linesDoneX.value[i]][j]=0
    }
    score.value+=8
  }

  for(let i=0;i<linesDoneY.value.length;i+=1)
  {
    for(let j=0;j<matrix.value[0].length;j+=1)
    {
      matrix.value[j][linesDoneY.value[i]]=0
    }
    score.value+=8
  }

  linesDoneX.value = []
  linesDoneY.value =[]
})




document.addEventListener('DOMContentLoaded', ()=>{
  const game = document.querySelector('.game')
  const parts = document.querySelectorAll('.work-tiles')

  

  // Щас всё для DragAndDrop
  let isDragging = false
  let DraggingElement
  let offsetX
  let offsetY

  document.addEventListener('mousedown',(e)=>{
    parts.forEach(part => {
      if(e.target.parentNode.parentNode==part && !e.target.classList.contains('not-visible'))
      {
        isDragging = true
        DraggingElement = part
        offsetX = e.pageX - (part.getBoundingClientRect().x + window.scrollX)
        offsetY = e.pageY - (part.getBoundingClientRect().y + window.scrollY)
      }
    });
  })

  document.addEventListener('mousemove',(e)=>{
    if(isDragging==true){
      DraggingElement.style.left = e.pageX-offsetX+'px'
      DraggingElement.style.top = e.pageY-offsetY+'px'
    }

  })

  document.addEventListener('mouseup',(e)=>{
    if(isDragging){
    isDragging = false

    let gameX = game.getBoundingClientRect().x + window.scrollX
    let gameY = game.getBoundingClientRect().y + window.scrollY

    let dragX = DraggingElement.getBoundingClientRect().x + window.scrollX
    let dragY = DraggingElement.getBoundingClientRect().y + window.scrollY

    let iteration = 0
    let jteration = 0

    for(let y = gameY; y<gameY+gameSize; y+=tileSize)
    {
      if(dragY>y-offsetpixel && dragY<y+offsetpixel) break
      iteration+=1
    }

    for(let x = gameX; x<gameX+gameSize; x+=tileSize)
    {
      if(dragX>x-offsetpixel && dragX<x+offsetpixel) break
      jteration+=1
    }
    
    if(iteration<8 && jteration<8){
      if(8-iteration<partMatrix.value.length)
      {

      }
      else if(8-jteration<partMatrix.value[0].length)
      {

      }
      else{
        let flag=true
        for(let i=0;i<partMatrix.value.length;i+=1)
        {
          for(let j=0; j<partMatrix.value[i].length;j+=1)
          {
            if(matrix.value[iteration+i][jteration+j]>0 && partMatrix.value[i][j]>0)flag = false
          }
        }

        if(flag){
        for(let i=0;i<partMatrix.value.length;i+=1)
        {
          for(let j=0; j<partMatrix.value[i].length;j+=1)
          {
            if(matrix.value[iteration+i][jteration+j]<1 && partMatrix.value[i][j]>0) matrix.value[iteration+i][jteration+j] = 1
          }
        }
        setTimeout(()=> {
          partMatrix.value = matrixParts.value[Math.floor(Math.random()*10)]
        }, 100);
        
        }
      }
    }
  }
  })
})



</script>

<template>
  <div class="wrapper">
    <div class="score">{{ score }}</div>

    <div class="game">
      <RowTiles class="row"  v-for="tile in matrix" :row="tile" />
    </div>

    <div class="place-for-work-tiles">
      <WorkTiles class="work-tiles" :part="partMatrix" :tileSize="tileSize" :idPart="1" />
    </div>

  </div>
</template>

<style scoped>

  .wrapper{
    width: 100%;
    height: 100vh;
  }

.game{
  width: 600px;
  height: 600px;
  margin: 0 auto;
  margin-top: 100px;
  display: flex;
  flex-direction: column;
}

.row{
  width: 100%;
  height: 12.5%;
}

.place-for-work-tiles{
  width: 1000px;
  height: 400px;
  margin: 0 auto;
  margin-top: 20px;
  background-color: rgba(127, 255, 212, 0.135);
}

</style>
