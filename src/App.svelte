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

<header />
<main >
  <div class="conteudo">
    {#if erro}
      <div class="erro">
        Esta aplicação requer acesso ao microfone para funcionar.
      </div>
    {:else}
      <div class="controls">
        <button disabled={gravando} id="start" on:click={iniciaGravacao}
          >Gravar</button
        >
        <button disabled={!gravando} id="stop" on:click={paraGravacao}
          >Parar</button
        >
        {#if gravando}
          <p>Gravando</p>
        {:else}
          <p>Aguardando</p>
        {/if}
      </div>
    {/if}
  
    <ul class="gravacoes">
      {#each gravacoes as gravacao}
        <li>
          <p>Gravação {gravacao.id}</p>
          <!-- svelte-ignore a11y-media-has-caption -->
          <audio controls src={gravacao.audioURL} />
          <button on:click={() => excluiGravacao(gravacao)}>Excluir</button>
        </li>
      {/each}
    </ul>
  </div>
  <div>
    <isvg href="./img/ilustracao1.svg" alt="ilustration audio"/>
  </div>
</main>

<style>
 
  main {
    display: flex;
    justify-content: center;
  }
  .conteudo{
    width: 80%;
    margin: 5rem 0;
    text-align: center;
    background-color: #FFFAFF;
    color: #145C9E;
    border-radius: 1rem;
  }
  .controls{
    display: flex;
    justify-content: center;
    padding: 1rem 0;
  }
  
</style>
