<template>

  <button type="button" @click="vid_play_pause()">Play/Pause</button>
  <button type="button" @click="currentFrame()">frame?</button>
  <div class="container">
    <!-- <canvas class="canvas" id="myCanvas" width="1520" height="720"></canvas> -->
    <svg id="mySvg" width="1520" height="720">
    </svg>
    <video class="video" id="myVideo" src="../src/assets/20220408155705.webm" width="1520" height="720"></video>
  </div>
  
</template>

<script>
import csvFile from 'raw-loader!./assets/data_0408_152650.csv';
import * as d3 from 'd3'; 

export default {
  name: 'App',
  components: {
  },
  data() {
    return {
      myData: [],
      currentTime:0,
      width: 1520,
      height: 720,
      maxNum: 0,
      gridX:32,
      gridY:16,
    }
  },
  created() {
    this.myData = csvFile
  },
  methods: {
    vid_play_pause: () =>{
      let myVideo = document.getElementById("myVideo"); 
      if (myVideo.paused) { 
        myVideo.play(); 
        } else { 
          myVideo.pause();
        }

    },
    currentFrame: () => {
      let myVideo = document.getElementById("myVideo"); 
      let currentTime = myVideo.currentTime
      // console.log(currentTime)
        return currentTime
    },
    initPosInGrid(x,y) {
      let posInGrid = new Array(x);
      for(let i=0; i<x; i++){
        posInGrid[i] = new Array(y);
        for(let j=0; j<y; j++) {
          posInGrid[i][j] = 0;
        }
      }
      return posInGrid
    },
    findMax(posInGrid,x) {
      let tempMax=0;
      for(let i=0;i<x;i++){
        if(tempMax < Math.max(...posInGrid[i])){
          tempMax = Math.max(...posInGrid[i])
        }
      }
      return tempMax
    },
    calPosInGrid(posInGrid,x,y,length,data,width,height) {
      for(let i =0; i<length; i++){
        if(data[i].length >=4){
          for(let j=4; j<data[i].length; j+=3){
            for(let a=0; a<x; a++){
              for(let b=0; b<y; b++){
                // console.log(data[i][j])
                if((data[i][j]>=a*width/x)&&(data[i][j]<(a+1)*width/x)){
                  if((data[i][j+1] >= b*height/y) && (data[i][j+1] < (b+1)*height/y)){
                    posInGrid[a][b] = posInGrid[a][b] + 1
                  }
                }
              }
            }
          }
        }
      }
      console.log(posInGrid)
      return posInGrid
    }
  },
  watch: {
   currentTime() {

    //  console.log(this.currentTime)
   }
  },
  mounted() {
    setInterval(()=>{
      this.currentTime = this.currentFrame()
    },200)

    let dataArray1 = new Array
    let dataArray2 = new Array
    dataArray1 = this.myData.split('\n')
    for (let i = 0; i < dataArray1.length; i++) {
      dataArray2[i] = dataArray1[i].split(',')
    }
    const mySvg = d3.select("#mySvg").style("border-width", "0.5px");

    mySvg.append("rect")
      .attr("x", 0)
      .attr("y", 0)
      .attr("width", this.width)
      .attr("height", this.height)
      .style("fill", "150")
      .attr("opacity", 0.5)

    for (let i = 0; i < this.gridX; i++) {
      mySvg.append("line")
        .attr("x1", i*this.width/this.gridX)
        .attr("y1", 0)
        .attr("x2", i*this.width/this.gridX)
        .attr("y2", this.height)
        .style("stroke", "grey")
        .style("stroke-width", "1px")
    }
    for (let i = 0; i < this.gridY; i++) {
      mySvg.append("line")
        .attr("x1", 0)
        .attr("y1", i*this.height/this.gridY)
        .attr("x2", this.width)
        .attr("y2", i*this.height/this.gridY)
        .style("stroke", "grey")
        .style("stroke-width", "1px")
    }
    let posInGrid = this.initPosInGrid(this.gridX,this.gridY)
    posInGrid = this.calPosInGrid(posInGrid,this.gridX,this.gridY,dataArray1.length,dataArray2,this.width,this.height)
    this.maxNum = this.findMax(posInGrid,this.gridX)
    
    for(let i=0; i<this.gridX; i++){
      for(let j=0; j<this.gridY; j++){
        mySvg.append("rect")
          .attr("x",i*this.width/this.gridX)
          .attr("y",j*this.height/this.gridY)
          .attr("width", this.width/this.gridX)
          .attr("height", this.height/this.gridY)
          .style("fill", "red")
          .style("opacity", posInGrid[i][j]/this.maxNum*100)
          .style("mix-blend-mode", "multiply")
        console.log(posInGrid[i][j]/this.maxNum)
      }
    }

    
  },
    

}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.container {
    position: relative;
  }
#mySvg {
  position: absolute;
  top:0;
  left:0;
  z-index: 10;
}
.video {
  position: absolute;
  top: 0;
  left: 0;
}
/* .canvas {
  position: absolute;
  top: 0;
  left: 0;
  z-index: 10;
  background-color:rgba(70, 40, 151, 0.5);
} */
</style>
