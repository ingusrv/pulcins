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
        dienas: string[];
    };

    type SkolenuDati = {
        grupas: string[];
        skoleni: Skolens[];
    };

    let dati: Dati;
    let grupas: string[] = [];
    let skoleni: Skolens[] = [];
    let newGroupDate: string;
    let currentGrupa: string;
    let currentDate: string;
    $: loadDataFromDate(currentDate);
    let skolensName: string = "";

    const storedItem = window.localStorage.getItem("pulcins");
    if (storedItem) {
        const parsed = JSON.parse(storedItem);
        console.log("datu versija", parsed.versija);
        // migrācija
        if (parsed.versija === 1) {
            dati = {
                versija: 2,
                dienas: [],
            };
            grupas = parsed.grupas;
            skoleni = parsed.skoleni;
        } else {
            dati = {
                versija: parsed.versija,
                dienas: parsed.dienas,
            };
        }

        currentDate = dati.dienas[dati.dienas.length - 1];
    } else {
        console.log(
            "Skolenu saraksts netika atrasts! Tiek izveidots jauns saraksts.",
        );
        dati = {
            versija: 2,
            dienas: [],
        };
    }

    function loadDataFromDate(date: string) {
        console.log(date);
        if (!date) {
            console.log("datums nav pareizs");
            return;
        }
        const d = new Date(date);
        const item = localStorage.getItem(`pulcins=${d.toDateString()}`);
        if (!item) {
            console.error(`datumā ${date} netika atrasti dati`);
            return;
        }

        const parsed = JSON.parse(item) as SkolenuDati;
        grupas = parsed.grupas;
        skoleni = parsed.skoleni;
        currentGrupa = parsed.grupas[0];
    }

    function createNewDay() {
        if (!newGroupDate) {
            alert("Datums nevar būt tukšs");
            return;
        }

        // pēdējās saglabātās dienas datums
        const d = new Date(dati.dienas[dati.dienas.length - 1]);
        currentDate = d.toDateString();

        // dienas datums, kur jaunie dati tiks saglabāti
        const dc = new Date(newGroupDate);
        dati.dienas = [...dati.dienas, dc.toDateString()];
        currentDate = dc.toDateString();
        skoleni.forEach((s) => (s.ieradies = false));
        skoleni = skoleni;

        storeData();
    }

    function storeData() {
        let v: SkolenuDati = {
            grupas: grupas,
            skoleni: skoleni,
        };

        const d = new Date(currentDate);
        localStorage.setItem(`pulcins=${d.toDateString()}`, JSON.stringify(v));

        localStorage.setItem("pulcins", JSON.stringify(dati));
        console.log("dati saglabāti");
    }

    function save() {
        storeData();
        window.alert("Dati saglabāti");
    }

    function addGroup() {
        if (!currentDate) {
            alert("Lūdzu vispirms pievienojiet jaunu dienu");
            return;
        }

        const groupName = prompt("Grupas nosaukums:");
        if (!groupName) {
            alert("Grupas nosaukums nedrīkst būt tukšs!");
            return;
        }

        grupas = [...grupas, groupName];
        currentGrupa = groupName;
    }

    function deleteGroup() {
        const groupName = prompt("Ievadi grupas nosakumu");
        if (groupName == null) {
            window.alert("Grupas nosaukums nedrīkst būt tukšs");
            return;
        }

        grupas = grupas.filter((name) => name !== groupName);
        skoleni = skoleni.filter((skolens) => skolens.grupa !== groupName);
        if (currentGrupa === groupName) {
            currentGrupa = grupas[0];
        }
        console.log(grupas);
        console.log(skoleni);

        window.alert("Grupa izdēsta");
    }

    function addSkolens() {
        if (!currentDate) {
            alert("Lūdzu vispirms pievienojiet jaunu dienu");
            return;
        }

        if (!currentGrupa) {
            alert("Lūdzu vispirms pievienojiet jaunu grupu");
            return;
        }

        if (!skolensName) {
            alert("Skolēna vārds nevar būt tukšs");
            return;
        }

        console.log("jauns skolēns:", skolensName, "grupā:", currentGrupa);
        const skolens: Skolens = {
            id: window.crypto.randomUUID(),
            vards: skolensName,
            grupa: currentGrupa,
            ieradies: false,
            iesniegums: false,
            komplektaId: 0,
        };
        skoleni = [...skoleni, skolens];
        skolensName = "";
    }

    function setIeradies(id: string) {
        skoleni.forEach((skolens) => {
            if (skolens.id === id) {
                skolens.ieradies = !skolens.ieradies;
                console.log(`${skolens.vards} ieradies: ${skolens.ieradies}`);
            }
        });
        skoleni = skoleni;
    }

    function setIesniegums(id: string) {
        skoleni.forEach((skolens) => {
            if (skolens.id === id) {
                skolens.iesniegums = !skolens.iesniegums;
                console.log(
                    `${skolens.vards} iesniegums: ${skolens.iesniegums}`,
                );
            }
        });
        skoleni = skoleni;
    }

    function deleteSkolens(id: string) {
        skoleni = skoleni.filter((skolens) => skolens.id !== id);
        console.log("skolēns izdzēsts");
    }
</script>

<main>
    <div class="controls">
        <button on:click={save}>Saglabāt</button>
        <button on:click={addGroup}>Jauna grupa</button>
        <button on:click={deleteGroup}>Dzēst grupu</button>
        <div class="line">
            <input type="date" bind:value={newGroupDate} />
            <button on:click={createNewDay}>Jauna diena</button>
        </div>
        <div class="line">
            <input
                type="text"
                placeholder="Vārds Uzvārds"
                bind:value={skolensName}
            />
            <button on:click={addSkolens}>Pievienot</button>
        </div>
        <div class="line">
            <select name="date" id="date" bind:value={currentDate}>
                {#each dati.dienas as diena}
                    <option value={diena}>
                        {new Date(diena).toLocaleDateString("lv")}
                    </option>
                {/each}
            </select>
            <select name="group" id="group" bind:value={currentGrupa}>
                {#each grupas as grupa}
                    <option value={grupa}>{grupa}</option>
                {/each}
            </select>
        </div>
    </div>
    <div class="grid">
        {#each skoleni as skolens}
            {#if skolens.grupa === currentGrupa}
                <div class="skolens" class:ieradies={skolens.ieradies}>
                    {skolens.vards}
                    <div class="actions">
                        <button
                            class="ir"
                            on:click={() => setIeradies(skolens.id)}
                        >
                            {skolens.ieradies ? "Ir" : "Nav"}
                        </button>
                        <button
                            class="lapina"
                            class:lap={skolens.iesniegums}
                            on:click={() => setIesniegums(skolens.id)}
                        >
                            {skolens.iesniegums
                                ? "Ir iesniegums"
                                : "Nav iesniegums"}
                        </button>
                        <input
                            class="robotId"
                            type="number"
                            bind:value={skolens.komplektaId}
                        />
                        <button
                            class="delete"
                            on:click={() => deleteSkolens(skolens.id)}
                        >
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
                                <path
                                    stroke="none"
                                    d="M0 0h24v24H0z"
                                    fill="none"
                                />
                                <path d="M4 7l16 0" />
                                <path d="M10 11l0 6" />
                                <path d="M14 11l0 6" />
                                <path
                                    d="M5 7l1 12a2 2 0 0 0 2 2h8a2 2 0 0 0 2 -2l1 -12"
                                />
                                <path
                                    d="M9 7v-3a1 1 0 0 1 1 -1h4a1 1 0 0 1 1 1v3"
                                />
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
        display: flex;
        gap: 1rem;
        flex-wrap: wrap;
        margin-top: 1rem;
    }

    @media screen and (max-width: 500px) {
        .skolens {
            width: 100%;
        }

        .line {
            width: 100%;
        }
    }

    .controls {
        display: flex;
        flex-direction: row;
        flex-wrap: wrap;
        gap: 0.3rem;
        row-gap: 0.5rem;
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
