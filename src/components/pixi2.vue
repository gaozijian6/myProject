<template>
    <div class="main">
          <canvas ref="pixiContainer"></canvas>
          <button class="left" @click="left"></button>
          <button class="right" @click="right"></button>
          <!-- <button @click="generate" class="add">add</button> -->
          <button class="start" @click="start">开始</button>
          <div class="score">{{ score }}</div>
    </div>
  </template>
  
  <script>
  import * as PIXI from 'pixi.js';
  import gsap from 'gsap';
  
  export default {
  name:'pixi2',
  data() {
      return {
          centerX:500,
          centerY:500,
          side1:100,
          side2:307.8461,
          side3:446.4102,
          green:'0x2ECC71',
          blue:'0x3498DB',
          red:'0xE74C3C',
          yellow:'0xF1C40F',
          hexagon3:null,
          hexagon2:null,
          hexagon1:null,
          isLeft:true,
          isRight:true,
          crash:[0,0,0,0,0,0],
          inside:[[],[],[],[],[],[]],
          itemArr:[],
          app:new PIXI.Application(),
          v:1,
          interval:3000,
          color:['0xE74C3C','0xF1C40F','0x3498DB','0x2ECC71'],
          score:0,
        //   记录消失方块
          record1:[],
        //   记录移动的方块
          record2:[],
          flag:false,
          timer:null
      }
  },
  computed: {
      diso_init(){
          return (this.side3/2)*Math.sqrt(3)
      }
  },
  mounted() {
      this.app = new PIXI.Application({
        width: 1800,
        height: 1000,
        transparent: false,
        antialias: true,
        backgroundColor: 0x1099bb,
        view: this.$refs.pixiContainer
      });
      const hexagon3 = new PIXI.Graphics()
      const hexagon2 = new PIXI.Graphics()
      const hexagon1 = new PIXI.Graphics()
      let points1=[]
      let points2=[]
      let points3=[]
      hexagon3.beginFill(0xffffff)
      for(let i=0;i<6;i++)
      {
          let deg=60*i
          let rad=Math.PI/180*deg
          const point_x = this.centerX + this.side3 * Math.cos(rad);
          const point_y = this.centerY + this.side3 * Math.sin(rad);
          points3.push(point_x, point_y);
      }
      hexagon3.drawPolygon(points3);
      hexagon3.endFill();
      this.app.stage.addChild(hexagon3)
      this.hexagon3=hexagon3
     
      hexagon2.beginFill(0xBDC3C7)
      for(let i=0;i<6;i++)
      {
          let deg=60*i
          let rad=Math.PI/180*deg
          const point_x = this.centerX + this.side2 * Math.cos(rad);
          const point_y = this.centerY + this.side2 * Math.sin(rad);
          points2.push(point_x, point_y);
      }
      hexagon2.drawPolygon(points2);
      hexagon2.endFill();
      this.app.stage.addChild(hexagon2)
      this.hexagon2=hexagon2
  
      hexagon1.beginFill(0xF1C40F)
      for(let i=0;i<6;i++)
      {
          let deg=60*i
          let rad=Math.PI/180*deg
          const point_x = this.side1 * Math.cos(rad);
          const point_y = this.side1 * Math.sin(rad);
          points1.push(point_x, point_y);
      }
      hexagon1.drawPolygon(points1);
      hexagon1.endFill();
      this.app.stage.addChild(hexagon1)
      this.hexagon1=hexagon1
      hexagon1.interactive=true
      hexagon1.position.set(500,500)
  },
  methods: {
    start(){
        this.timer=setInterval(() => {
            this.generate(this.v)
        }, this.interval);
    },
      left(){
          if(this.isLeft){
              this.isLeft=false
              gsap.to(this.hexagon1, {rotation: this.hexagon1.rotation - Math.PI / 3, duration: 0.1});
              let val=this.crash.shift()
              this.crash.push(val)
              let arr=this.inside.shift()
              this.inside.push(arr)
              for(let obj of this.itemArr)
              {
                  if(obj.isFinish){
                      gsap.to(obj.item, {rotation: obj.item.rotation - Math.PI / 3, duration: 0.1});
                  }
              }
              setTimeout(() => {
                  this.isLeft=true
              }, 100);
          }
      },
      right(){
          if(this.isRight){
              this.isRight=false
              gsap.to(this.hexagon1, {rotation: this.hexagon1.rotation + Math.PI / 3, duration: 0.1});
              let val=this.crash.pop()
              this.crash.unshift(val)
              let arr=this.inside.pop()
              this.inside.unshift(arr)
              for(let obj of this.itemArr)
              {
                  if(obj.isFinish){
                      gsap.to(obj.item, {rotation: obj.item.rotation + Math.PI / 3, duration: 0.1});
                  }
              }
              setTimeout(() => {
                  this.isRight=true
              }, 100);
          }
      },
      generate(v){
          let item=new PIXI.Graphics()
          let polygon1=new PIXI.Polygon([-this.side3/2,0,this.side3/2,0,this.side3/2-10*Math.sqrt(3),30,10*Math.sqrt(3)-this.side3/2,30,])
          let color=this.color[Math.floor(Math.random()*4)]
          item.beginFill(color)
          item.drawPolygon(polygon1)
          item.endFill()
          let type=Math.floor(Math.random()*6)
          item.position.set(500,500)
          item.pivot.set(0, this.side3 / 2 * Math.sqrt(3))
          switch(type){
              case 0:
                  break
              case 1:
                  item.rotation+=Math.PI/3
                  break
              case 2:
                  item.rotation+=Math.PI/3*2
                  break
               case 3:
                  item.rotation+=Math.PI/3*3
                  break
              case 4:
                  item.rotation+=Math.PI/3*4
                  break
              case 5:
                  item.rotation+=Math.PI/3*5
                  break
          }
          this.app.stage.addChild(item)
          let obj={item,type,isFinish:'false',color,advance:0}
          this.itemArr.push(obj)
          this.move(v)
      },
      move(v){
          let ticker=new PIXI.Ticker()
          let obj=this.itemArr.pop()
          let {item,type,color,isFinish,advance}=obj
          item.clear()
          let polygon1=new PIXI.Polygon([-this.side3/2,0,this.side3/2,0,this.side3/2-10*Math.sqrt(3),30,10*Math.sqrt(3)-this.side3/2,30,])
          item.beginFill(color)
          item.drawPolygon(polygon1)
          item.endFill()
          let y=this.side3 / 2 * Math.sqrt(3)
          let ratio
          ticker.add(()=>{
              y-=v
              ratio=y/this.diso_init
              item.clear()
              let polygon1=new PIXI.Polygon([-this.side3/2*ratio,0,this.side3/2*ratio,0,(this.side3/2)*ratio-10*Math.sqrt(3),30,10*Math.sqrt(3)-this.side3/2*ratio,30,])
              item.beginFill(color)
              item.drawPolygon(polygon1)
              item.endFill()
              item.pivot.set(0,y)
              if(y<=50*Math.sqrt(3)+30*(this.crash[type]+1)){
                  y=50*Math.sqrt(3)+30*(this.crash[type]+1)
                  item.pivot.set(0,y)
                  ratio=y/this.diso_init
                  item.clear()
                  let polygon1=new PIXI.Polygon([-this.side3/2*ratio,0,this.side3/2*ratio,0,(this.side3/2)*ratio-10*Math.sqrt(3),30,10*Math.sqrt(3)-this.side3/2*ratio,30,])
                  item.beginFill(color)
                  item.drawPolygon(polygon1)
                  item.endFill()
                  isFinish=true
                  this.itemArr.push({item,type,color,isFinish})
                  this.crash[type]++
                  this.inside[type].push({item,type,color,isFinish,advance})
                  this.Check(type,this.inside[type].length-1)
                  ticker.stop()
              }
          })
          ticker.start()
      },
      render(){
        let isFirst=false
        if(!this.flag){
           this.flag=true
           isFirst=true
        }
          for(let i=0;i<6;i++)
          {
              for(let j=0;j<this.inside[i].length;j++)
              {
                  let target
                  if(this.inside[i][j].advance){
                      let y=this.inside[i][j].item.pivot.y
                      target=y-this.inside[i][j].advance*30
                      gsap.to(this.inside[i][j].item.pivot, { duration: 0.2, y: target })
                      let dis=y-target
                      let forward=dis/12
                      let t=0
                      let polygon=new PIXI.Polygon()
                      gsap.to(polygon,{duration:0.2,onUpdate:()=>{
                      t+=forward
                      let ratio=y/this.diso_init
                      polygon.points=[-this.side3/2*ratio+t/2,0,this.side3/2*ratio-t/2,0,(this.side3/2)*ratio-10*Math.sqrt(3)-t/2,30,10*Math.sqrt(3)-this.side3/2*ratio+t/2,30,]
                      this.inside[i][j].item.clear()
                      this.inside[i][j].item.beginFill(this.inside[i][j].color)
                      this.inside[i][j].item.drawPolygon(polygon)
                      this.inside[i][j].item.endFill()
                      
                      
                  }})
                  setTimeout(() => {
                    let ratio=target/this.diso_init
                    this.inside[i][j].item.clear()
                    let polygon1=new PIXI.Polygon([-this.side3/2*ratio,0,this.side3/2*ratio,0,(this.side3/2)*ratio-10*Math.sqrt(3),30,10*Math.sqrt(3)-this.side3/2*ratio,30,])
                    this.inside[i][j].item.beginFill(this.inside[i][j].color)
                    this.inside[i][j].item.drawPolygon(polygon1)
                    this.inside[i][j].item.endFill()
                    this.app.stage.addChild(this.inside[i][j].item)
                    this.inside[i][j].item.pivot.y=target
                    this.inside[i][j].advance=0
                  }, 200);
                  
                  }
              }
          }
          this.score+=this.record1.length
          if(this.score>=10&&this.score<30&&this.score-this.record1.length<10){
            clearInterval(this.timer)
            this.v+=0.25
            this.interval-=300
            this.timer=setInterval(() => {
                this.generate(this.v)
            }, this.interval);
          }
          else if(this.score>=30&&this.score<50&&this.score-this.record1.length<30){
            clearInterval(this.timer)
            this.v+=0.25
            this.interval-=300
              this.timer=setInterval(() => {
                this.generate(this.v)
            }, this.interval);
          }
          else if(this.score>=50&&this.score<80&&this.score-this.record1.length<50){
            clearInterval(this.timer)
            this.v+=0.25
            this.interval-=300
              this.timer=setInterval(() => {
                this.generate(this.v)
            }, this.interval);
          }
          else if(this.score>=80&&this.score-this.record1.length<80){
            clearInterval(this.timer)
            this.v+=0.25
            this.interval-=300
              this.timer=setInterval(() => {
                this.generate(this.v)
            }, this.interval);
          }
          this.record1=[]
          if (isFirst) {
                  setTimeout(() => {
                    for (let i = 0; i < 6; i++) {
                      for (let j = 0; j < this.inside[i].length; j++) {
                          this.Check(i, j)
                      }
                  }
                  this.flag=false
                  }, 500);
              }
          
        },
      Check(type,j){
        // 记录最低点
        let arr1=[110,110,110,110,110,110]
        // 记录每组需要删除数量
        let arr2=[0,0,0,0,0,0]
        this.check(type,j)
        if(this.record1.length<3){
            this.record1=[]
            return
        }
        for(let {x,y} of this.record1)
        {
            // this.inside[x][y].item
            this.app.stage.removeChild(this.inside[x][y].item)
            // this.inside[x][y].item.destory()
            arr1[x]=arr1[x]<y?arr1[x]:y
            arr2[x]++
            this.crash[x]--
        }
        for(let x=0;x<6;x++)
        {
            if(arr1[x]<6){
                this.inside[x].splice(arr1[x],arr2[x])
                for(let y=arr1[x];y<this.inside[x].length;y++)
                {
                    this.inside[x][y].advance+=arr2[x]
                }
            }
        }
        this.render()
      },
      check(type,j){
        if(!this.isHave({x:type,y:j})){
            this.record1.push({x:type,y:j})
        }
        let len=this.inside[type].length
        let left=(type-1+6)%6
        let right=(type+1)%6
        let down=j-1>=0?j-1:null
        let up=j+1<len?j+1:null
        let left_len=this.inside[left].length
        let right_len=this.inside[right].length
        let color=this.inside[type][j].color
        
        if(left_len>j&&this.inside[left][j].color==color&&!this.isHave({x:left,y:j})){
            this.record1.push({x:left,y:j})
            this.check(left,j)
        }      
        if(right_len>j&&this.inside[right][j].color==color&&!this.isHave({x:right,y:j})){
            this.record1.push({x:right,y:j})
            this.check(right,j)
        }  
        if(down!=null&&this.inside[type][down].color==color&&!this.isHave({x:type,y:down})){
            this.record1.push({x:type,y:down})
            this.check(type,down)
        }
        if(up&&this.inside[type][up].color==color&&!this.isHave({x:type,y:up})){
            this.record1.push({x:type,y:up})
            this.check(type,up)
        }
      },
      isHave(obj){
        if(!this.record1.length)return false
        for(let e of this.record1)
        {
            if(e.x==obj.x&&e.y==obj.y)return true
        }
        return false
      }

  },
  }
  </script>
  
  <style>
  canvas{
      position: absolute;
  }
  button{
      width: 30px;
      height: 30px;
      z-index: 10;
      border: none;
      position: absolute;
  }
  button:hover{
      cursor: pointer;
  }
  .left{
      left: 420px;
      top: 485px;
      clip-path: polygon(0 50%,100% 0,100% 100%);
  }
  .right{
      left: 550px;
      top: 485px;
      clip-path: polygon(0 0,0 100%,100% 50%);
  }
  .add{
      
  }
  .score{
      position: absolute;
      /* background-color: #b10808; */
      width: 100px;
      height: 50px;
      left: 450px;
      top: 475px;
      font-size: 40px;
      text-align: center;
      line-height: 40px;
  
  }
  .start{
    width: 100px;
    height: 30px;
    position: absolute;
    z-index: 10;
  }
  
  </style>