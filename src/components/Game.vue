<script>
export default {
  data() {
    return {
      playerPosition: 0,
      playerSpeed: 10, 
      isJumping: false,
      jumpHeight:100,
    };
  },
  mounted() {
    // Chiamare la funzione per dare il movimento all'avvio
    this.movePlayer(0); // 0 indica che l'elemento non si muove all'avvio

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
    movePlayerUp(){
      var currentY = parseInt(getComputedStyle(document.getElementById('player')).bottom);
      var newY = currentY + this.jumpHeight;
      document.getElementById('player').style.bottom= newY + 'px';
    },
    movePlayerDown(){
      document.getElementById('player').style.bottom= '0px';
    }
  },
};
</script>
<template lang="">
  <div class="game-bg">
    <div id="player" :style="{ left: playerPosition + 'px' }"></div>
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
</style>