<script lang="ts">
  import { onMount } from "svelte";
  interface Character {
    version: number;
    name: string;
    hp: {
      current: string;
      total: string;
    };
    silver: string;
    description: string;
    stats: {
      str: string;
      tou: string;
      pres: string;
      agi: string;
    };
    weapon1: { die: string; name: string };
    weapon2: { die: string; name: string };
    armour: { die: string; name: string };
    items: { name: string }[];
    powers: { name: string }[];
    powerUses: number;
    omens: number;
  }

  let character: Character;
  let loading = false;
  let loadingCancel;

  let currentItem;
  let currentPower;

  const dice = ["", "d2", "d4", "d6", "d8", "d10", "d12"];
  const damageReductionDice = ["", "-d2", "-d4", "-d6"];
  const damageReductionDiceAdditional = {
    "-d4": "+2 DR Agility tests, +2 DR Defence tests",
    "-d6": "+4 DR Agility tests, +2 DR Defence tests",
  };
  const persist = (): void => {
    loading = true;
    if (loadingCancel) {
      clearTimeout(loadingCancel);
    }
    localStorage.setItem("character", JSON.stringify(character));
    loadingCancel = setTimeout(() => (loading = false), 400);
  };

  const clear = (): void => {
    localStorage.removeItem("character");
    character = getCharacter();
  };

  const addPower = () => {
    character.powers = [...character.powers, { name: currentPower }];
    currentPower = "";
    persist();
  };

  const addItem = () => {
    character.items = [...character.items, { name: currentItem }];
    currentItem = "";
    persist();
  };

  const removePower = (power: { name: string }) => {
    const index = character.powers.indexOf(power);
    if (index === -1) {
      return;
    }
    character.powers.splice(index, 1);
    character.powers = [...character.powers];
    persist();
  };

  const removeItem = (item: { name: string }) => {
    const index = character.items.indexOf(item);
    if (index === -1) {
      return;
    }
    character.items.splice(index, 1);
    character.items = [...character.items];
    persist();
  };

  function getCharacter(): Character {
    return (
      JSON.parse(localStorage.getItem("character")) || {
        version: 1,
        name: "",
        hp: {
          current: "",
          total: "",
        },
        silver: "",
        description: "",
        stats: {
          str: "",
          tou: "",
          pres: "",
          agi: "",
        },
        weapon1: { die: "", name: "" },
        weapon2: { die: "", name: "" },
        armour: { die: { additional: "", die: "" }, name: "" },
        items: [],
        powers: [],
        powerUses: 0,
        omens: 0,
      }
    );
  }

  // Lifecycle
  onMount(async () => {
    loading = true;
    character = getCharacter();
    loadingCancel = setTimeout(() => (loading = false), 400);
  });
</script>

<main>
  <section class="flex space-between">
    <h1>Mörk Borg: Character Sheet</h1>

    <!-- By Sam Herbert (@sherb), for everyone. More @ http://goo.gl/7AJzbL -->
    <svg
      class:hidden={!loading}
      class="pink"
      width="38"
      height="38"
      viewBox="0 0 38 38"
      xmlns="http://www.w3.org/2000/svg"
      stroke="currentColor"
    >
      <g fill="none" fill-rule="evenodd">
        <g transform="translate(1 1)" stroke-width="2">
          <circle stroke-opacity=".5" cx="18" cy="18" r="18" />
          <path d="M36 18c0-9.94-8.06-18-18-18">
            <animateTransform
              attributeName="transform"
              type="rotate"
              from="0 18 18"
              to="360 18 18"
              dur="1s"
              repeatCount="indefinite"
            />
          </path>
        </g>
      </g>
    </svg>
  </section>
  {#if character}
    <div>
      <!-- Top Banner -->
      <section class="flex distribute">
        <article class="form-group">
          <label for="name">Name</label>
          <input
            bind:value={character.name}
            on:blur={persist}
            name="name"
            id="name"
            type="text"
            autocapitalize="none"
            autocorrect="off"
          />
        </article>
        <article class="form-group">
          <label for="hp">HP</label>
          <input
            bind:value={character.hp.current}
            on:blur={persist}
            class="short-input"
            type="text"
            pattern="-?[0-9]+"
            name="hp"
            id="hp"
            autocapitalize="none"
          />
          <strong>/</strong>
          <input
            bind:value={character.hp.total}
            on:blur={persist}
            class="short-input white-form"
            name="max-hp"
            id="max-hp"
            autocapitalize="none"
            type="text"
            pattern="[0-9]+"
          />
          <label for="max-hp">(max)</label>
        </article>

        <article class="form-group">
          <label for="silver">Silver</label>
          <input
            bind:value={character.silver}
            on:blur={persist}
            class="silver"
            name="silver"
            id="silver"
            autocapitalize="none"
            type="text"
            pattern="[0-9]*"
          />
        </article>

        <button class="link-button form-group" on:click={clear}>Clear</button>
        <article class="full-width form-group">
          <label for="description">Description</label>
          <textarea
            rows="5"
            id="description"
            bind:value={character.description}
            on:blur={persist}
          />
        </article>
      </section>

      <section>
        <h2><span class="ability-scores">Ability Scores</span></h2>
        <div class="flex distribute">
          <div class="form-group">
            <label for="strength">Strength</label>
            <input
              class="short-input pink-form"
              name="strength"
              id="strength"
              autocapitalize="none"
              type="text"
              pattern="(-?[0-6])?"
              bind:value={character.stats.str}
              on:blur={persist}
            />
          </div>
          <div class="form-group">
            <label for="agility">Agility</label>
            <input
              class="short-input pink-form"
              name="agility"
              id="agility"
              autocapitalize="none"
              type="text"
              pattern="(-?[0-6])?"
              bind:value={character.stats.agi}
              on:blur={persist}
            />
          </div>
          <div class="form-group">
            <label for="presence">Presence</label>
            <input
              class="short-input pink-form"
              name="presence"
              id="presence"
              autocapitalize="none"
              type="text"
              pattern="(-?[0-6])?"
              bind:value={character.stats.pres}
              on:blur={persist}
            />
          </div>
          <div class="form-group">
            <label for="toughness">Toughness</label>
            <input
              class="short-input pink-form"
              name="toughness"
              id="toughness"
              autocapitalize="none"
              type="text"
              pattern="(-?[0-6])?"
              bind:value={character.stats.tou}
              on:blur={persist}
            />
          </div>
        </div>
      </section>

      <!-- Weapons / Armor -->
      <section>
        <h2><span class="yellow-black">Weapons / Armour</span></h2>
        <div class="flex distribute">
          <div class="form-group">
            <label for="weapon-1">Main&nbsp;Weapon</label>
            <input
              name="weapon-1"
              id="weapon-1"
              autocapitalize="none"
              autocorrect="off"
              bind:value={character.weapon1.name}
              on:blur={persist}
            />

            <label for="weapon-1-dice">Dice</label>
            <select
              id="weapon-1-dice"
              bind:value={character.weapon1.die}
              on:blur={persist}
            >
              {#each dice as die}
                <option value={die}>
                  {die}
                </option>
              {/each}
            </select>
          </div>
          <div class="form-group">
            <label for="weapon-2">Off&nbsp;&nbsp;Weapon</label>
            <input
              name="weapon-2"
              id="weapon-2"
              autocapitalize="none"
              autocorrect="off"
              bind:value={character.weapon2.name}
            />

            <label for="weapon-2-dice">Dice</label>
            <select
              id="weapon-2-dice"
              bind:value={character.weapon2.die}
              on:blur={persist}
            >
              {#each dice as die}
                <option value={die}>
                  {die}
                </option>
              {/each}
            </select>
          </div>
          <div class="form-group">
            <label for="armour">Armour</label>
            <input
              name="armour"
              id="armour"
              autocapitalize="none"
              autocorrect="off"
              bind:value={character.armour.name}
              on:blur={persist}
            />

            <label for="armour-dice">Dice</label>
            <select
              id="armour-dice"
              class="mb-1"
              bind:value={character.armour.die}
              on:blur={persist}
            >
              {#each damageReductionDice as die}
                <option value={die}>
                  {die}
                </option>
              {/each}
            </select>
            {#if character.armour?.die && damageReductionDiceAdditional[character.armour?.die]}
              <p class="yellow dice-text">
                {damageReductionDiceAdditional[character.armour?.die]}
              </p>
            {/if}
          </div>
        </div>
      </section>
      <!-- Container for columns -->
      <div class="flex">
        <section class="flex column">
          <article>
            <h2><span class="pink-black">Equipment</span></h2>
            <p>
              Strength + 8 items, +2 DR on Agility/Strength tests when over
              burdened, 2x Strength+8 is max limit
            </p>
            <!-- TODO CLASS THAT MAKES THIS RED WHEN THE TEST ABOVE IS UNSATISFIED -->
            <p class="dice-text">{character.items.length} slots used</p>
            <div class="form-group mt-1">
              <label for="new-item">Item</label>
              <input
                class="white-form"
                name="new-item"
                id="new-item"
                autocapitalize="none"
                autocorrect="off"
                type="text"
                on:blur={persist}
                bind:value={currentItem}
              />
              <button class="link-button yellow" on:click={addItem}>Add</button>
            </div>
            <ol>
              {#each character.items as item}
                <li>
                  {item.name}
                  <button class="link-button" on:click={() => removeItem(item)}
                    >Remove</button
                  >
                </li>
              {/each}
            </ol>
          </article>
        </section>

        <section class="flex column">
          <article>
            <h2><span class="white-black">Powers</span></h2>
            <header>
              Presence + <span class="dice-text">d4</span> times per day.
              <div class="form-group mt-1">
                <label for="power-uses">Power Uses</label>
                <input
                  class="white-form short-input"
                  class:red-form={character.powerUses === 0}
                  name="power-uses"
                  id="power-uses"
                  autocapitalize="none"
                  autocorrect="off"
                  type="text"
                  on:blur={persist}
                  bind:value={character.powerUses}
                />

                <button
                  class="link-button"
                  on:click={() =>
                    (character.powerUses = Math.max(
                      character.powerUses - 1,
                      0
                    ))}>Use</button
                >
              </div>
            </header>
            <aside>
              Test presence D12. Fail lose <span class="dice-text">d2</span> HP,
              become dizzy for 1hr.
            </aside>
            <div class="form-group mt-1">
              <label for="new-power">Power</label>
              <input
                class="white-form"
                name="new-power"
                id="new-power"
                autocapitalize="none"
                autocorrect="off"
                type="text"
                on:blur={persist}
                bind:value={currentPower}
              />
              <button class="link-button pink" on:click={addPower}>Add</button>
            </div>
            <ol>
              {#each character.powers as power}
                <li>
                  {power.name}
                  <button
                    class="link-button"
                    on:click={() => removePower(power)}>Remove</button
                  >
                </li>
              {/each}
            </ol>
          </article>
        </section>

        <section class="flex column">
          <article>
            <h2><span class="pink-white">Omens</span></h2>
            <span class="dice-text">d2</span>
            <aside>
              <p>You can use your omens to:</p>
              <ul>
                <li>Deal maximum damage</li>
                <li>Reroll (yours or someone else's roll)</li>
                <li>Reduce damage by <span class="dice-text">d6</span></li>
                <li>Reduce a DR by -4</li>
                <li>Stop a crit/fumble</li>
              </ul>
            </aside>
            <div class="form-group mt-1">
              <label for="omen">Count</label>
              <input
                class="short-input white-form"
                type="text"
                pattern="[0-9]+"
                name="omen"
                id="omen"
                autocapitalize="none"
                on:blur={persist}
                bind:value={character.omens}
              />
            </div>
          </article>
        </section>
      </div>
    </div>
  {/if}
</main>

<style>
  main {
    display: flex;
    min-width: 100vw;
    flex-direction: column;
  }

  .flex {
    display: flex;
    flex-wrap: wrap;
  }

  .column {
    flex-direction: column;
    flex-basis: 100%;
  }

  .distribute {
    justify-content: space-between;
  }

  .space-between {
    justify-content: space-between;
  }

  @media (min-width: 850px) {
    .distribute {
      justify-content: space-around;
    }
  }

  @media (min-width: 850px) {
    .column {
      flex-basis: 33%;
    }
  }

  .short-input {
    max-width: 2rem;
  }

  .silver {
    max-width: 3.5rem;
  }

  .full-width {
    flex-basis: 100%;
  }

  .form-group {
    margin-bottom: 1rem;
  }

  .yellow-white {
    background-color: var(--yellow);
    color: var(--white);
  }

  .yellow-black {
    background-color: var(--yellow);
    color: var(--black);
  }

  .pink-white {
    background-color: var(--pink);
    color: var(--wrhite);
  }

  .pink-black {
    background-color: var(--pink);
    color: var(--black);
  }

  .white-black {
    background-color: var(--white);
    color: var(--black);
  }

  .ability-scores {
    letter-spacing: 2px;
  }
</style>
