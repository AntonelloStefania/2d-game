<script>
import levels from '../../levels.json'
export default {
  data() {
    return {
      playerPosition: 0,
      playerSpeed: 10, 
      isJumping: false,
      jumpHeight:100,
      obstacleWidth:50,
      obstacleHeigth:80,
      obstacles:[],
      holes:[],
      holeWidth:40,
      holeHeigth:200,
      maxObstacles: 4,
      playerWidth:100,
      gameOver:false,
      currentLevel:1,
      endLevel: window.innerWidth,
      levels: levels,
    };
  },
  mounted() {
    // Chiamare la funzione per dare il movimento all'avvio
     this.movePlayer(0); // 0 indica che l'elemento non si muove all'avvio
     this.initializeGame();
    // console.log(this.obstacles)
    // Aggiungere un ascoltatore di eventi per rilevare i tasti direzionali sinistro e destro
    document.addEventListener('keydown', (event) => {
      if (event.key === 'ArrowLeft' || event.key === 'a' || event.key === 'A') {
        // Muovi il player verso sinistra
        this.movePlayer(-this.playerSpeed);
      } else if (event.key === 'ArrowRight'|| event.key === 'd' || event.key === 'D') {
        // Muovi il player verso destra
        this.movePlayer(this.playerSpeed);
      } else if(event.key === ' '  && !this.isJumping){
        this.startJump()
      }
      
    });

    //verifico collisione ogni 100ms
   setInterval(() => {
      this.updateGame();
    }, 100);
  },
  methods: {
    movePlayer(direction) {
      // Calcola la nuova posizione in base alla direzione
      this.playerPosition += direction;

      // Assicurati che l'elemento "player" non esca dalla larghezza del contenitore
      const gameContainerWidth = document.querySelector('.game-bg').offsetWidth;
      const playerWidth = document.getElementById('player').offsetWidth;
      if (this.playerPosition < 0) {
        this.playerPosition = 0;
      }
      //  else if (this.playerPosition + playerWidth > gameContainerWidth) {
      //   this.playerPosition = gameContainerWidth - playerWidth;
      else if(this.playerPosition >= this.endLevel ){
        console.log('hai finito il livello')
        this.nextLevel()
      }
    },
      //FUNZIONA
    startJump(){
      this.isJumping = true;
      this.movePlayerUp();
      setTimeout(()=>{
        this.endJump();
      }, 500)
    },



    endJump(){
      this.isJumping= false;
      this.movePlayerDown();
    },
    //funzione che determina il salto
    movePlayerUp(){
      var currentY = parseInt(getComputedStyle(document.getElementById('player')).bottom);
      var newY = currentY + this.jumpHeight;
      document.getElementById('player').style.bottom= newY + 'px';
      this.playerSpeed = 100
      
    },
    //riporta player a terra
    movePlayerDown(){
      document.getElementById('player').style.bottom= '0px';
      this.playerSpeed = 10
    },

    //genera ostacoli ad inizio partita
    initializeGame(){
  
      var maxPositionX = window.innerWidth;
      var NumberObstacles =  this.levels.find(level => level.level === this.currentLevel).maxObstacles;
      var NumberHoles =  this.levels.find(level => level.level === this.currentLevel).maxHoles;
      const safeZoneWidth = 180;
     
      //OSTACOLI
      var obstacleWrapper= document.querySelector('.obstacle-wrapper');
      obstacleWrapper.innerHTML=''
      for(let i=1; i<=NumberObstacles; i++){
        const obstacle = document.createElement('div');
        obstacle.className = 'obstacle';
        obstacle.style.left = `${Math.random() * 100}%`;
       const randomPositionX = Math.floor(Math.random() * (maxPositionX - this.obstacleWidth - 2*safeZoneWidth )+ safeZoneWidth);
       obstacle.style.left = randomPositionX + 'px';
      this.obstacles.push({
        positionX: randomPositionX,
      });
      obstacleWrapper.appendChild(obstacle);
      }

      //BUCHI
      
      const visualHoleWidth = 130; // Larghezza visiva del buco (compresi i bordi di 10px da entrambi i lati)
      const tolerance = (visualHoleWidth - this.holeWidth) / 2; // Tolleranza equamente distribuita su entrambi i lati
      var holeWrapper = document.querySelector('.hole-wrapper');
      for (let i = 1; i <= NumberHoles; i++) {
        const hole = document.createElement('div');
        hole.className = 'hole';
        
        // Genera una posizione casuale con tolleranza
        const holeRandomPositionX = Math.floor(
          Math.random() * (maxPositionX - visualHoleWidth - 2 * safeZoneWidth) +
            safeZoneWidth
        );

      
       // hole.style.left = holeRandomPositionX - tolerance + 'px';

        // Imposta la larghezza visiva del buco
        hole.style.width = visualHoleWidth + 'px';
        
        this.holes.push({
          positionX: holeRandomPositionX ,
        });
         
      
        hole.style.left = holeRandomPositionX  - tolerance + 'px';

        holeWrapper.appendChild(hole);
      }
    },

    updateGame() {
  // Verifica collisioni
    let isColliding = false;
    this.obstacles.forEach((obstacle) => {
      //console.log('Player:', this.playerPosition, this.playerWidth);
     // console.log('Obstacle:', obstacle.positionX, this.obstacleWidth);
      var playerBottom = parseInt(getComputedStyle(document.getElementById('player')).bottom);

      if (
        this.playerPosition + this.playerWidth >= obstacle.positionX &&
        this.playerPosition <= obstacle.positionX + this.obstacleWidth &&
        playerBottom <= this.obstacleHeigth
      ){
       
        isColliding = true;
      
      }
    });
    console.log(isColliding);
    if (isColliding) {
      this.gameOver = true;
      this.playerSpeed=0;
      this.jumpHeight=0;
    
      
    }


    //PROVA COLLISIONE BUCHI
    let isCollidingHole = false;
  this.holes.forEach((hole) => {
    const playerBottom = parseInt(getComputedStyle(document.getElementById('player')).bottom);
    
     // Regola questo valore in base alle tue esigenze
  if (
    this.playerPosition + this.playerWidth >= hole.positionX &&
    this.playerPosition <= hole.positionX + this.holeWidth &&
    playerBottom <= this.holeHeigth 
  ) {
    isCollidingHole = true;
  }
  console.log(isCollidingHole)
});

  // Se c'è una collisione con un buco, aggiorna la posizione del giocatore solo se non sta saltando
  if (isCollidingHole && !this.isJumping) {
    // Imposta la nuova posizione verticale del giocatore (esempio: bottom: -170px)
    const newBottomPosition = -170;
    document.getElementById('player').style.bottom = newBottomPosition + 'px';
  }
},
restart(){
  this.playerPosition=0;
  this.playerSpeed=10;
  this.obstacles=[];
  this.holes=[];
  this.currentLevel=1;
  this.gameOver= false;
  this.jumpHeight=100;
  this.initializeGame();
},

//funzione passaggio a livello successivo
nextLevel() {
    // Incrementa il livello corrente
    this.currentLevel++;

    // Reimposta il giocatore e inizializza il nuovo livello
    this.playerPosition = 0;
    this.playerSpeed = 10;
    this.obstacles = [];
    this.gameOver = false;
    this.initializeGame();
  },

}

};
</script>
<template lang="">
  <div class="game-bg">
    <div id="player" :style="{ left: playerPosition + 'px' }"></div>
    <div class="obstacle-wrapper col-8 offset-2"></div>
    <div class="hole-wrapper"></div>
    <div class="game-over" v-if="gameOver">
      <h1>Game-Over</h1>
    <button class="btn btn-warning" @click="restart()">restart</button>
    </div>
    <div>
      <h2>Livello: {{currentLevel}}</h2>
    </div>
  </div>
</template>
<style lang="scss">
  .game-bg{
    width: 100vw;
    height:80vh;
    background-color: rgb(184, 184, 255);
    position:relative;
  }
  #player{
    width: 100px;
    height: 190px;
    background-color: red;
    position:absolute;
    bottom:0;
  }

  .obstacle{
    background-color: brown;
    width: 50px;
    height: 80px;
    position:absolute;
    bottom:0
  }

  .game-over{
    position: absolute;
    left:50%;
    top:50%;
    transform: translate(-50%, -50%);
    text-align: center;
  }
  body::-webkit-scrollbar {
  display: none;
}
.hole{
  background-color: red;
    position: absolute;
    bottom:-180px;
    
    width: 80px;
    height: 180px;
}
</style>