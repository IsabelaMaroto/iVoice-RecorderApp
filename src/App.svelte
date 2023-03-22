<script>
  let mediaRecorder = null;
  let gravando = false;
  let contador = 0;
  let gravacoes = [];
  let erro = false;

  function iniciaGravacao() {
    //chama a API só quando a função é chamada no botão
    navigator.mediaDevices.getUserMedia({ audio: true }).then(
      (stream) => {
        mediaRecorder = new MediaRecorder(stream);
        let chunks = [];

        mediaRecorder.ondataavailable = function (e) {
          chunks.push(e.data);
        };

        mediaRecorder.onstop = function (e) {
          const blob = new Blob(chunks, { type: "audio/ogg; codecs=opus" }); //type padrão;
          chunks = [];
          const audioURL = URL.createObjectURL(blob);
          gravacoes = [
            ...gravacoes,
            {
              id: ++contador,
              audioURL: audioURL,
            },
          ]; //svelte precisa que seja feito assim, com push não funciona!
          stream.getTracks().forEach((track) => track.stop()); //fecha o stream, cada track tem uma função stop;
        };

        gravando = true;
        mediaRecorder.start();
      },
      () => {
        erro = true;
      }
    );
  }

  function paraGravacao() {
    gravando = false;
    mediaRecorder.stop();
    mediaRecorder = null; //volta para null, pois estou criando um novo toda vez que inicia;
  }

  function excluiGravacao(gravacao) {
    gravacoes = gravacoes.filter((item) => item !== gravacao);
  }
</script>

<header>
  <div class="logo__box">
    <img src="../logo.png" alt="logo iVoice"/>
  </div>
  <div class="links">
    <p>Sobre nós</p>
    <p>Contatos</p>
    <p>Parceiros</p>
  </div>
</header>
<main>
  <div class="gravador">
    {#if erro}
      <div class="erro">
        <h2>Esta aplicação requer acesso ao microfone para funcionar!</h2>
        <div class="background-microfone">
          <img class="microfone" src="../voice_1.svg" alt="microfone" />
        </div>
      </div>
    {:else}
      <div class="status">
        {#if gravando}
          <div class="boxContainer">
            <div class="box box1" />
            <div class="box box2" />
            <div class="box box3" />
            <div class="box box4" />
            <div class="box box5" />
          </div>
        {:else}
          <div class="boxContainer">
            <div class="box " />
            <div class="box " />
            <div class="box " />
            <div class="box " />
            <div class="box " />
          </div>
        {/if}
      </div>
      <div class="controls">
        <div>
          <img
            disabled={gravando}
            id="start"
            on:click={iniciaGravacao}
            src="../play.svg"
            alt="play"
          />
        </div>

        <div class="white__background">
          <img
            disabled={!gravando}
            id="stop"
            on:click={paraGravacao}
            src="../stop.svg"
            alt="stop"
          />
        </div>
      </div>
    {/if}
  </div>

  <div class="lista__gravacoes">
    <ul class="gravacoes">
      {#each gravacoes as gravacao}
        <li>
          <p>Gravação {gravacao.id}</p>
          <!-- svelte-ignore a11y-media-has-caption -->
          <audio controls src={gravacao.audioURL} />
          <img
            on:click={() => excluiGravacao(gravacao)}
            src="../trash-bin.svg"
            alt="lixo"
          />
        </li>
      {/each}
    </ul>
  </div>

  <div class="tutorial">
    <div class="tutorial__top">
      <div class="conteudo">
        <h2>Grave os seus audios gratuitamente!</h2>
        <p>No gravador acima precione o botão de play e comece a grava já! Lembre-se que é necessário autorizar o navegador a utilizar o seu microfone!</p>
      </div>
      <div class="img__container">
        <img src="../img1.svg" alt="recorder girl" />
      </div>
    </div>
    <div class="tutorial__bottom">
      <div class="img__container">
        <img src="../img2.svg" alt="play girls" />
      </div>
      <div class="conteudo">
        <h2>Ouça e faça download dos seu audios!</h2>
        <p>Quando você parar o áudio ele será adicionado na sua lista de garvações, onde será possível ouvir, excluir ou baixar seus áudios em formato mp4!</p>
      </div>
    </div>
  </div>
</main>

<style>

/* Header */
  header {
    display: flex;
    justify-content: space-between;
    padding: 0 3rem;
    background-color: #fffaff;
  }
  header .logo__box img{
    height: 60px;
  }
  header .links{
    display: flex;
    justify-content: space-between;
    align-items: center;
    width: 400px;
    color: #F15025;
    font-size: 1rem;
    font-weight: bold;
  }
  header .links p{
    border-bottom: 1px solid #fffaff;
  }
  
  header .links p:hover{
    cursor: pointer;
    border-bottom: 1px solid #145C9E;
  }
main {
  overflow-x: hidden;
  display: flex;
  flex-direction: column;
  align-items: center;
  margin: 0 auto;
  width: 80%;
}

/* gravador */
  .gravador {
    display: flex;
    flex-direction: column;
    align-items: center;
    width: 500px;
    height: 200px;
    background-color: #FFFAFF;
    margin: 2rem 0;
    border-radius: 1rem;
    padding: 1rem 0;
    box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2), 0 6px 20px 0 rgba(0, 0, 0, 0.19);
  }
  .controls {
    display: flex;
    justify-content: center;
  }
  .controls div {
    padding: 0 1rem;
  }
  .controls img {
    height: 50px;
  }
  .controls img:hover {
    cursor: pointer;
  }
/* svg background */
  .white__background {
    position: relative;
  }
  .white__background img {
    position: relative;
    z-index: 2;
  }
  .white__background::before {
    content: "";
    position: absolute;
    width: 50%;
    height: 48%;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    background-color: #fffaff;
    z-index: 1;
  }

/* microfone icon */
  .microfone {
    height: 100px;
    position: relative;
    z-index: 2;
  }
  .background-microfone {
    position: relative;
  }
  .background-microfone::before {
    content: "";
    position: absolute;
    width: 60%;
    height: 70%;
    top: 30%;
    left: 50%;
    transform: translate(-50%, -30%);
    background-color: #fffaff;
    z-index: 1;
  }

/* erro */
  .erro {
    display: flex;
    flex-direction: column;
    align-items: center;
    text-align: center;
    padding: 2rem 1rem;
  }
  .erro h2 {
    margin-bottom: 1rem;
  }
/* status */
  .status {
    padding: 2rem 0;
  }

/* lista de gravações */
  .lista__gravacoes {
    background-color: #30BCED;
    border-radius: 1rem;
    width: 500px;
    margin-bottom: 2rem;
  }
  .lista__gravacoes li {
    list-style: none;
    display: flex;
    justify-content: space-around;
    align-items: center;
    margin: 1rem 0;
  }
  .lista__gravacoes li img {
    height: 50px;
  }
  .lista__gravacoes li img:hover {
    cursor: pointer;
  }
  .lista__gravacoes li p {
    font-size: 1rem;
    font-weight: bold;
  }



/* tutorial */
  .tutorial .tutorial__top, .tutorial__bottom{
    display: flex;
    justify-content: space-around;
    align-items: center;
    margin: 7rem 0;
  }
  .conteudo{
    width: 30%;
  }
  .conteudo h2{
    margin-bottom: 3rem;
    font-size: 2rem;
  }
  .img__container img{
    height: 400px;
  }



/* animation */
  .boxContainer {
    display: flex;
    justify-content: space-between;
    height: 80px;
    --boxSize: 10px;
    --gutter: 4px;
    width: calc((var(--boxSize) + var(--gutter)) * 5);
  }

  .box {
    transform: scaleY(0.4);
    height: 100%;
    width: var(--boxSize);
    background: rgb(48, 188, 237);
    background: linear-gradient(
      0deg,
      rgba(48, 188, 237, 1) 0%,
      rgba(241, 80, 37, 1) 97%
    );
    animation-duration: 1.2s;
    animation-timing-function: ease-in-out;
    animation-iteration-count: infinite;
    border-radius: 8px;
  }

  .box1 {
    animation-name: quiet;
  }

  .box2 {
    animation-name: normal;
  }

  .box3 {
    animation-name: quiet;
  }

  .box4 {
    animation-name: loud;
  }

  .box5 {
    animation-name: quiet;
  }
  @keyframes quiet {
    25% {
      transform: scaleY(0.6);
    }
    50% {
      transform: scaleY(0.4);
    }
    75% {
      transform: scaleY(0.8);
    }
  }

  @keyframes normal {
    25% {
      transform: scaleY(1);
    }
    50% {
      transform: scaleY(0.4);
    }
    75% {
      transform: scaleY(0.6);
    }
  }
  @keyframes loud {
    25% {
      transform: scaleY(1);
    }
    50% {
      transform: scaleY(0.4);
    }
    75% {
      transform: scaleY(1.2);
    }
  }

  
</style>
