<script lang="ts">
  type Skolens = {
    id: string;
    vards: string;
    grupa: string;
    ieradies: boolean;
    iesniegums: boolean;
    komplektaId: number;
  };

  type Dati = {
    versija: number;
    grupas: string[];
    skoleni: Skolens[];
  };

  let dati: Dati;
  let skoleni: Skolens[] = [];
  const storedItem = window.localStorage.getItem("pulcins");
  if (storedItem) {
    dati = JSON.parse(storedItem);
    console.log("datu versija", dati.versija);
    skoleni = dati.skoleni;
  } else {
    console.log(
      "Skolenu saraksts netika atrasts! Tiek izveidots jauns saraksts.",
    );

    dati = {
      versija: 1,
      grupas: [],
      skoleni: [],
    };
  }

  function save() {
    dati.skoleni = skoleni;
    window.localStorage.setItem("pulcins", JSON.stringify(dati));
    console.log("saved");
  }

  function load() {
    const storedItem = window.localStorage.getItem("pulcins");
    if (storedItem) {
      dati = JSON.parse(storedItem);
      console.log("datu versija", dati.versija);
      skoleni = dati.skoleni;
    } else {
      alert("Skolēnu saraksts netika atrasts!");
    }
    console.log("loaded");
  }

  function addGroup() {
    const groupName = prompt("Grupas nosaukums:");
    if (!groupName) {
      alert("Grupas nosaukums nedrīkst būt tukšs!");
      return;
    }

    dati.grupas = [...dati.grupas, groupName];
    currentGrupa = groupName;
  }

  let currentGrupa: string = dati.grupas[0];
  let skolensName: string = "";
  function addSkolens() {
    console.log("adding new name:", skolensName, "to group:", currentGrupa);
    const skolens: Skolens = {
      id: window.crypto.randomUUID(),
      vards: skolensName,
      grupa: currentGrupa,
      ieradies: false,
      iesniegums: false,
      komplektaId: 0,
    };
    skoleni = [...skoleni, skolens];
  }

  function setIeradies(id: string) {
    console.log("setting ir for", id);
    skoleni.forEach((skolens) => {
      if (skolens.id === id) {
        skolens.ieradies = !skolens.ieradies;
      }
    });
    skoleni = skoleni;
  }

  function setIesniegums(id: string) {
    console.log("setting lap for", id);
    skoleni.forEach((skolens) => {
      if (skolens.id === id) {
        skolens.iesniegums = !skolens.iesniegums;
      }
    });
    skoleni = skoleni;
  }

  function deleteSkolens(id: string) {
    console.log("deleting skolens with id", id);
    skoleni = skoleni.filter((skolens) => skolens.id !== id);
  }
</script>

<main>
  <div class="controls">
    <button on:click={save}>Saglabāt</button>
    <button on:click={load}>Ielādēt</button>
    <select name="group" id="group" bind:value={currentGrupa}>
      {#each dati.grupas as grupa}
        <option value={grupa}>{grupa}</option>
      {/each}
    </select>
    <button on:click={addGroup}>Jauna grupa</button>
    <input type="text" placeholder="Vārds Uzvārds" bind:value={skolensName} />
    <button on:click={addSkolens}>Pievienot</button>
  </div>

  <div class="grid">
    {#each skoleni as skolens}
      {#if skolens.grupa === currentGrupa}
        <div class="skolens" class:ieradies={skolens.ieradies}>
          {skolens.vards}
          <div class="actions">
            <button class="ir" on:click={() => setIeradies(skolens.id)}>
              {skolens.ieradies ? "Ir" : "Nav"}
            </button>
            <button
              class="lapina"
              class:lap={skolens.iesniegums}
              on:click={() => setIesniegums(skolens.id)}
            >
              {skolens.iesniegums ? "Ir iesniegums" : "Nav iesniegums"}
            </button>
            <input
              class="robotId"
              type="number"
              bind:value={skolens.komplektaId}
            />
            <button class="delete" on:click={() => deleteSkolens(skolens.id)}>
              <svg
                xmlns="http://www.w3.org/2000/svg"
                class="icon icon-tabler icon-tabler-trash"
                width="24"
                height="24"
                viewBox="0 0 24 24"
                stroke-width="2"
                stroke="currentColor"
                fill="none"
                stroke-linecap="round"
                stroke-linejoin="round"
              >
                <path stroke="none" d="M0 0h24v24H0z" fill="none" />
                <path d="M4 7l16 0" />
                <path d="M10 11l0 6" />
                <path d="M14 11l0 6" />
                <path d="M5 7l1 12a2 2 0 0 0 2 2h8a2 2 0 0 0 2 -2l1 -12" />
                <path d="M9 7v-3a1 1 0 0 1 1 -1h4a1 1 0 0 1 1 1v3" />
              </svg>
            </button>
          </div>
        </div>
      {/if}
    {/each}
  </div>
</main>

<style>
  .grid {
    display: grid;
    grid-template-columns: 1fr 1fr 1fr;
    gap: 1rem;
    margin-top: 1rem;
  }

  .controls button {
    background-color: var(--secondary);
    color: var(--on-secondary);
    border: none;
    border-radius: 6px;
    padding: 0.25rem 0.5rem;
  }

  .skolens {
    background-color: var(--danger);
    color: var(--on-danger);

    padding: 1rem;
    border-radius: 12px;
    font-size: 1.5rem;
  }

  .lapina {
    background-color: red;
    background-color: #ea2b1f;
  }

  .lap {
    background-color: green;
    background-color: #008148;
  }

  .ieradies {
    background-color: var(--success);
    color: var(--on-success);
  }

  .actions {
    display: flex;
    gap: 0.2rem;
    margin-top: 1rem;
  }

  .actions button {
    border: none;
    border-radius: 6px;
  }

  .ir {
    width: 2rem;
    height: 2rem;
  }

  .robotId {
    width: 3rem;
    border: none;
    border-radius: 6px;
  }

  .delete {
    color: black;
    background: none;
  }
</style>
