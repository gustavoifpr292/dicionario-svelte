<script>
  let palavra = "";
  let resultado = null;
  let erro = null;
  let carregando = false;

  async function buscar() {
    if (!palavra.trim()) return;

    resultado = null;
    erro = null;
    carregando = true;

    try {
      const res = await fetch(`https://api.dicionario-aberto.net/word/${palavra}`);
      if (!res.ok) throw new Error("Erro ao consumir API");

      const data = await res.json();

      if (data.length === 0) {
        erro = "Nenhum resultado encontrado.";
        return;
      }

      // api retornando em xml, converter pra texto
      const xml = data[0].xml;
      const parser = new DOMParser();
      const doc = parser.parseFromString(xml, "text/xml");

      let defs = [...doc.getElementsByTagName("def")].map(d => d.textContent.trim());

      resultado = defs.join("\n\n");
    } catch (e) {
      erro = e.message;
    } finally {
      carregando = false;
    }
  }
</script>

<style>
  body {
    font-family: sans-serif;
  }
  .card {
    max-width: 600px;
    margin: auto;
    padding: 20px;
  }
  textarea {
    width: 100%;
    min-height: 150px;
  }
</style>

<div class="card">
  <h1>Dicionário PT-BR</h1>

  <input
    placeholder="Digite uma palavra, letra ou fonema..."
    bind:value={palavra}
    on:keydown={(e) => e.key === 'Enter' && buscar()}
  />

  <button on:click={buscar}>Buscar</button>

  {#if carregando}
    <p>Carregando...</p>
  {/if}

  {#if erro}
    <p style="color: red;">{erro}</p>
  {/if}

  {#if resultado}
    <h2>Significado:</h2>
    <textarea readonly>{resultado}</textarea>
  {/if}
</div>
