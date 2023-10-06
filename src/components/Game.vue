<script>
import levels from '../../levels.json'
import spriteData from '../../spritesheet.json';
import spriteIdle from '../../spritesheet-idle.json';
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
      isRunning:false,
    };
  },
  mounted() {
    // Chiamare la funzione per dare il movimento all'avvio
     this.movePlayer(0); // 0 indica che l'elemento non si muove all'avvio
     this.initializeGame();
    // console.log(this.obstacles)

    //FUNZIONA <--------------------------------------------------------------------------------------------------------------------------------------------------
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
      // Aggiungi un ascoltatore di eventi per il rilascio dei tasti direzionali
  document.addEventListener('keyup', (event) => {
    if (
      (event.key === 'ArrowLeft' || event.key === 'a' || event.key === 'A' || 
      event.key === 'ArrowRight' || event.key === 'd' || event.key === 'D') && 
      !this.isJumping
    ) {
      // Rilascio del pulsante direzionale, impostare direction su 0
      this.movePlayer(0);
    }
  });
  // FINE FUNZIONANTE <-----------------------------------------------------------------------------------------------------------------------------------------------


    //verifico collisione ogni 100ms
   setInterval(() => {
      this.updateGame();
    }, 100);
  },
  methods: {
    movePlayer(direction) {
      // Calcola la nuova posizione in base alla direzione
      this.playerPosition += direction;
      console.log(direction)
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
       // Se il giocatore si muove verso destra, imposta l'animazione di corsa
       const playerElement = document.getElementById('player');
      if (direction > 0) {
        // Muovi il giocatore verso destra
        playerElement.style.transform = 'scaleX(1)'; // Imposta la direzione del giocatore
        this.isRunning = true; // Aggiungi classe di animazione
      } else if (direction < 0) {
        // Muovi il giocatore verso sinistra
        playerElement.style.transform = 'scaleX(-1)'; // Imposta la direzione del giocatore
        this.isRunning = true; // Aggiungi classe di animazione
      } else if (direction==0) { 
        // Il giocatore è fermo NON CI ENTRO MAI, NON RIESCO A PASSARE DA RUN A IDLE
        this.isRunning = false;
        document.getElementById('player').classList.remove('run-animation');
        document.querySelector('.scia').classList.add('d-none');
        document.getElementById('player').classList.add('idle-sprite');
      }
      console.log(this.isRunning)
    },
 

      //FUNZIONA
    startJump(){
      this.isJumping = true;
      this.movePlayerUp();
      document.getElementById('player').classList.add('player-jumping'); // Aggiungi la classe

      setTimeout(()=>{
        this.endJump();
      }, 500)
    },



    endJump(){
      this.isJumping= false;
      document.getElementById('player').classList.remove('player-jumping'); // Rimuovi la classe

      this.movePlayerDown();
    },
    //funzione che determina il salto
    movePlayerUp(){
      var currentY = parseInt(getComputedStyle(document.getElementById('player')).bottom);
      var newY = currentY + this.jumpHeight;
      document.getElementById('player').style.bottom= newY + 'px';
      
      this.playerSpeed = 120
      if ( this.playerSpeed == 120) {
        document.querySelector('.scia').classList.remove('d-none');
      } 
       
      
    },
    //riporta player a terra
    movePlayerDown(){
      
      document.getElementById('player').style.bottom= '0px';
      document.querySelector('.scia').classList.add('d-none');
      this.playerSpeed = 10
      this.isRunning= false
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
        const holeImage = document.createElement('img');
         holeImage.src = `../../leaves.png`; // Assumi che le immagini si chiamino hole1.png, hole2.png, ecc.
         hole.appendChild(holeImage);
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
   // console.log(isColliding);
    if (isColliding) {
      this.gameOver = true;
      this.playerSpeed=0;
      this.jumpHeight=0;
      document.getElementById('player').classList.remove('player-jumping');
      document.getElementById('player').classList.add('player-death');

    
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
 // console.log(isCollidingHole)
});

  // Se c'è una collisione con un buco, aggiorna la posizione del giocatore solo se non sta saltando
  if (isCollidingHole && !this.isJumping) {
    // Imposta la nuova posizione verticale del giocatore (esempio: bottom: -170px)
    const newBottomPosition = -170;
    this.gameOver = true;
      this.playerSpeed=0;
      this.jumpHeight=0;
      document.getElementById('player').classList.remove('player-jumping');
      document.getElementById('player').classList.add('player-death');


     document.getElementById('player').style.bottom = newBottomPosition + 'px';
  }
},
restart(){
  const newBottomPosition =0;
  document.getElementById('player').style.bottom = newBottomPosition + 'px';
  this.playerPosition=0;
  this.playerSpeed=10;
  document.getElementById('player').classList.remove('player-death');
  const holeWrapper = document.querySelector('.hole-wrapper');
 
  holeWrapper.innerHTML = '';
  this.obstacles=[];
  this.holes=[];
  this.currentLevel=1;
  const backgroundImage = `../../${this.levels.find(level => level.level === this.currentLevel).backgroundImage}`;
    document.documentElement.style.setProperty('--background-image', `url(${backgroundImage})`);
  this.gameOver= false;
  this.jumpHeight=100;
  this.initializeGame();
},

//funzione passaggio a livello successivo
nextLevel() {
    // Incrementa il livello corrente
    this.currentLevel++;
    const holeWrapper = document.querySelector('.hole-wrapper');
      holeWrapper.innerHTML = '';
    // Reimposta il giocatore e inizializza il nuovo livello
    this.playerPosition = 0;
    this.playerSpeed = 10;
    this.obstacles = [];
    this.holes=[];
    this.gameOver = false;
    
    const backgroundImage = `../../${this.levels.find(level => level.level === this.currentLevel).backgroundImage}`;
    document.documentElement.style.setProperty('--background-image', `url(${backgroundImage})`);
    console.log(backgroundImage)
    this.initializeGame();
  },

}

};
</script>
<template lang="">
  <div class="game-bg">
    <div id="player"  :style="{ left: playerPosition + 'px' }"  :class="{ 'run-animation': isRunning,'idle-animation': !isRunning }">
      <img src="../../scia.png" id="line" class="scia d-none"  style="left:0" alt="">
    </div>
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
    background-image: var(--background-image,url(bg-3.jpg));
    position:relative;
    background-position: center;
  }
  .run-animation {
    animation: run-animation 0.5s steps(9) infinite;
    background-image: url(spritesheet-run.png);
    background-repeat: no-repeat;
    /* background-position: -11px 100px; */
    background-size: 1113px 140px;
    
  }

  .idle-animation {
    animation: run-animation 0.8s steps(9) infinite;
    background-image: url(spritesheet-idle.png);
    background-repeat: no-repeat;
    /* background-position: -11px 100px; */
    background-size: 1113px 140px;
    transform:scaleX(0.7) !important;
  }

  .player-death{
   // animation: 2s steps(9) infinite;
  
    background-image: url(spritesheet-death.png);
    // background-repeat: no-repeat;

    /* background-position: -11px 100px; */
    background-size: 1113px 140px;
    transform:scaleX(1.2) !important;
  }



  //ANIMAZIONE DEL GLIDE MA FUNZIONA MALE
  // .glide{
  //   background-image: url(spritesheet-glide.png);
  //   background-repeat: no-repeat;
  //   /* background-position: -11px 100px; */
  //   background-size: 1113px 140px;
  //   transform:scaleX(0.7) !important;
  // }


/* Imposta l'immagine di sfondo del giocatore */
#player {
  width: 109px;
    height: 190px;
    position: absolute;
    bottom: 0;
    transition: bottom 0.2s ease; 
    z-index: 200;
  
}
.scia{
  width: 318px;
    position: relative;
    left: -211px !important;
    bottom: -15px !important;
}

@keyframes run-animation {
  0% {
    background-position:  0px 60px;;
  }
  100% {
    background-position: -1000px 60px; /* Imposta la posizione finale in base ai tuoi frame */
   
  }
}


  .obstacle{
   background-image: url(../../spritesheet-trap.png);
    width: 50px;
    height: 80px;
    position:absolute;
    bottom:0;
    animation:obstacle-animation 1s steps(9) infinite;

    /* background-position: -11px 100px; */
    background-size: 580px 140px;
    transform: scaleX(1) !important;
  }


  @keyframes obstacle-animation {
  0% {
    background-position: 0px 0px;
  }
  100% {
    background-position: -520px 0px; /* Verifica questo valore in base alla larghezza totale del tuo spritesheet */
  }
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
  background: rgb(36,21,12);
  background: linear-gradient(0deg, rgba(36,21,12,1) 35%, rgba(122,69,38,1) 76%, rgba(141,191,73,1) 92%); 
  position: absolute;
  bottom:-180px;
    
    width: 80px;
    height: 180px;
    img{
      width: 235px;
    bottom: 65px;
    position: absolute;
    right: -25px;
    }
}

// @keyframes jump-animation {
  
//   0% {
//     transform: translateY(0);
//   }
//   50% {
//     transform: translateY(-100px); // Altezza massima del salto
//   }
//   100% {
//     transform: translateY(0px);
//   }
// }

.player-jumping {
 // animation: jump-animation 0.5s  steps(9) ease-out; // Regola la durata e l'accelerazione dell'animazione
 // animation: run-animation 0.5s infinite;
    background-image: url(spritesheet-jump.png);
    background-repeat: no-repeat;
    /* background-position: -11px 100px; */
    background-size: 1113px 140px;
    transform: scaleX(1) !important;
}



</style>