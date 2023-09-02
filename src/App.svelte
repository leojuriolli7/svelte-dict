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

  const API_URL = "https://api.dictionaryapi.dev/api/v2/entries/en/";

  const debounce = (newValue: string) => {
    clearTimeout(timer);
    timer = setTimeout(() => {
      value = newValue;
    }, 750);
  };

  $: if (value) {
    fetch(API_URL + value).then(async (res) => {
      const response: Definition[] = await res.json();
      definitions = response;
    });
  }
</script>

<main>
  <div class="title">
    <BookMarked color="#5bc0be" size={24} />
    <h1 style:color="#5bc0be">Search the English Dictionary</h1>
  </div>

  <input
    type="text"
    class="search"
    placeholder="Type a word..."
    on:keyup={({ currentTarget: { value } }) => debounce(value)}
  />

  {#if value && definitions?.length}
    {#each definitions as definition}
      <p>{definition.word}</p>

      {#each definition.meanings as meaning}
        <p>{meaning.partOfSpeech}</p>
      {/each}
    {/each}
  {/if}
</main>

<style lang="scss">
  main {
    width: 100%;
    margin: 0 auto;
    padding: 5rem;
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
    color: #fff;
    margin-top: 12px;

    &::placeholder {
      color: rgb(69, 110, 163);
    }
  }
</style>
