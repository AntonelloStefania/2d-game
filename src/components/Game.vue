<script>
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
      maxObstacles: 4,
      playerWidth:100,
      gameOver:false,
    };
  },
  mounted() {
    // Chiamare la funzione per dare il movimento all'avvio
     this.movePlayer(0); // 0 indica che l'elemento non si muove all'avvio
     this.initializeGame();
     console.log(this.obstacles)
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
    const gameInterval = setInterval(() => {
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
      } else if (this.playerPosition + playerWidth > gameContainerWidth) {
        this.playerPosition = gameContainerWidth - playerWidth;
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
      var NumberObstacles = Math.floor(Math.random()* (4 - 1 +1))+1
      const safeZoneWidth = 180;
     
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

    },
    // Funzione di aggiornamento del gioco (chiamata ad intervalli regolari)
    // updateGame() {
    //   // Verifica la collisione tra il personaggio e gli ostacoli
    //   this.obstacles.forEach((obstacle) => {
    //     if (
    //       this.playerPosition + this.playerWidth >= obstacle.positionX &&
    //       this.playerPosition <= obstacle.positionX + obstacle.width
    //     ) {
    //       // Il personaggio è in contatto con l'ostacolo, blocca il movimento orizzontale
    //       this.playerSpeedX = 0;
    //     }
    //   });

    //   // Continua a muovere il personaggio se non c'è collisione
    //   if (this.playerSpeedX !== 0) {
    //     this.movePlayer(this.playerSpeedX);
    //   }
    // },
    updateGame() {
  // Verifica collisioni
    let isColliding = false;
    this.obstacles.forEach((obstacle) => {
      console.log('Player:', this.playerPosition, this.playerWidth);
      console.log('Obstacle:', obstacle.positionX, this.obstacleWidth);
      const playerBottom = parseInt(getComputedStyle(document.getElementById('player')).bottom);

      if (
        this.playerPosition + this.playerWidth >= obstacle.positionX &&
        this.playerPosition <= obstacle.positionX + this.obstacleWidth &&
        playerBottom <= this.obstacleHeigth
      ){
        // Il personaggio è in contatto con l'ostacolo, segna la collisione ma non bloccare il personaggio
        isColliding = true;
      
      }
    });
    console.log(isColliding);
    if (isColliding) {
      this.gameOver = true;
      this.playerSpeed=0;
    
      
    }
    console.log(this.gameOver)
  // Continua a muovere il personaggio se non c'è collisione
  // if (this.playerSpeed !== 0) {
  //   this.movePlayer(this.playerSpeed);
  // }
},
restart(){
  this.playerPosition=0;
  this.playerSpeed=10;
  this.obstacles=[];
  this.gameOver= false;
  this.initializeGame();
}

}

};
</script>
<template lang="">
  <div class="game-bg">
    <div id="player" :style="{ left: playerPosition + 'px' }"></div>
    <div class="obstacle-wrapper col-8 offset-2"></div>
    <div class="game-over" v-if="gameOver">
    <h1>Game-Over</h1>
    <button class="btn btn-warning" @click="restart()">restart</button>
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
</style>