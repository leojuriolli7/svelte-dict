<script lang="ts">
  import { BookMarked } from "lucide-svelte";

  type Definition = {
    word: string;
    phonetic: string;
    phonetics: {
      text: string;
      audio?: string;
    }[];
    origin?: string;
    meanings: {
      partOfSpeech: string;
      definitions: {
        definition: string;
        example: string;
        synonyms: string[];
        antonyms: string[];
      }[];
    }[];
  };

  let timer: number;
  let value: string;
  let definitions: Definition[];

  let isLoading = false;

  const API_URL = "https://api.dictionaryapi.dev/api/v2/entries/en/";

  const debounce = (newValue: string) => {
    clearTimeout(timer);
    timer = setTimeout(() => {
      value = newValue;
    }, 750);
  };

  $: if (value) {
    isLoading = true;
    fetch(API_URL + value)
      .then(async (res) => {
        const response: Definition[] = await res.json();
        definitions = response;
      })
      .finally(() => {
        isLoading = false;
      });
  }
</script>

<main>
  <div class="title">
    <BookMarked color="#fffffb" size={24} />
    <h1 style:color="#fffffb">Search the English Dictionary</h1>
  </div>

  <input
    type="text"
    class="search"
    placeholder="Type a word..."
    on:keyup={({ currentTarget: { value } }) => debounce(value)}
  />

  {#if isLoading}
    <p>Loading...</p>
  {/if}

  {#if value && !isLoading && !definitions?.length}
    <p>No results found for {value}</p>
  {/if}

  {#if value && definitions?.length && !isLoading}
    {#each definitions as definition}
      <div class="word-definition">
        <h2 class="word-text">{definition.word}</h2>

        {#if definition.phonetics.length}
          {#each definition.phonetics as phonetic}
            <div class="phonetic-container">
              {#if phonetic.text}
                <p class="word-phonetic">[ {phonetic.text} ]</p>
              {/if}

              {#if phonetic.audio}
                <audio src={phonetic.audio} controls />
              {/if}
            </div>
          {/each}
        {/if}

        {#each definition.meanings as meaning}
          <div class="meaning">
            <h3 class="part-of-speech">{meaning.partOfSpeech}</h3>

            <ul>
              {#each meaning.definitions as meaningDefinition}
                <li class="meaning-definition">
                  <p class="meaning-explanation">
                    {meaningDefinition.definition}
                  </p>

                  {#if meaningDefinition.example}
                    <p class="meaning-example">{meaningDefinition.example}</p>
                  {/if}
                </li>
              {/each}
            </ul>
          </div>
        {/each}
      </div>
    {/each}
  {/if}
</main>

<style lang="scss">
  main {
    padding: 5rem;

    @media (max-width: 1200px) {
      padding: 3rem;
    }

    @media (max-width: 800px) {
      padding: 1.5rem;
    }
  }

  .title {
    display: flex;
    align-items: center;
    gap: 15px;
  }

  input.search {
    padding: 12px;
    background-color: #3a506b;
    font-size: 16px;
    border-radius: 20px;
    border: none;
    color: #fffffb;
    margin-top: 12px;

    &::placeholder {
      color: rgb(69, 110, 163);
    }
  }

  .word-definition {
    padding: 24px;
    margin-top: 12px;
  }

  .word-text {
    font-size: 32px;
    position: sticky;
    top: 0;
    left: 0;
    background: #1c2541;
    z-index: 3;
  }

  .meaning {
    border-left: 2px solid #3a506b;
    padding-left: 10px;
  }

  .part-of-speech {
    padding: 12px 0;
    font-weight: 600;
    font-size: 20px;
    position: sticky;
    top: 40px;
    left: 0;
    background: #1c2541;
  }

  .phonetic-container {
    display: flex;
    align-items: center;
    gap: 12px;
    margin: 6px 0 18px 0;

    .word-phonetic {
      letter-spacing: 2px;
    }
  }

  .meaning-definition {
    margin: 0 0 24px 0;
  }

  .meaning-example {
    margin: 12px 0;
    quotes: "“" "”";
    padding: 12px;
    background: #3a506b;
    border-radius: 6px;
    width: fit-content;

    &::before {
      content: open-quote;
    }
    &::after {
      content: close-quote;
    }
  }
</style>
