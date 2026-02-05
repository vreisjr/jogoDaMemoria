<script lang="ts">
  import { goto } from '$app/navigation';
  let tituloDoJogo: string = "JOGO DA MEMÓRIA";
  console.log("tituloDoJogo:", tituloDoJogo);
</script>

<h1>
    {tituloDoJogo}
</h1>

<br />
<button class="menu" on:click={() => goto('/jogar')}>Jogar</button>
<br />
<a class="menu" href="/sobre">Sobre</a>
<br />
<a class="menu" href="/ajuda">Ajuda</a>