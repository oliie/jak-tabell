<script lang="ts">
  import { onMount } from 'svelte';

  const INITIAL_DATA = [['', '', '', '']];
  const INITIAL_HEADINGS = [
    'M3 Tabellnamn',
    'M3 Kolumnnamn',
    'Koppling till annan M3 Tabell',
    'Mätetal'
  ];

  let headings = structuredClone(INITIAL_HEADINGS);
  let data: string[][] = structuredClone(INITIAL_DATA);

  onMount(() => {
    const storedHeadings = localStorage.getItem('HEADINGS');
    const storedData = localStorage.getItem('DATA');

    if (!!storedHeadings) headings = JSON.parse(storedHeadings);
    if (!!storedData) data = JSON.parse(storedData);
  });

  $: isValid = data.every((row) =>
    row.every((_val, _i, arr) => {
      return !!arr[0] && !!arr[1];
    })
  );

  const storeData = () => {
    localStorage.setItem('HEADINGS', JSON.stringify(headings));
    localStorage.setItem('DATA', JSON.stringify(data));
  };

  const addRow = (index: number) => {
    data.splice(index + 1, 0, new Array(data[0].length).fill(null));
    data = data;
    storeData();
  };

  const removeRow = (index: number) => {
    if (confirm(`Vill du ta bort rad ${index + 1}?`)) {
      data.splice(index, 1);
      data = data;
      storeData();
    }
  };

  const addColumn = () => {
    data.forEach((row) => row.push(''));
    headings.push('');

    headings = headings;
    data = data;
    storeData();
  };

  const resetTable = () => {
    if (confirm('Vill du återställa tabellen?')) {
      headings = structuredClone(INITIAL_HEADINGS);
      data = structuredClone(INITIAL_DATA);
      storeData();
    }
  };

  const exportToExcel = () => {
    const filename = prompt('Ange filnamn...');

    // European Excel use ";" while American use ","
    const delimiter = ';';
    let output = 'data:text/csv;charset=utf-8,';

    output += headings.join(delimiter) + '\r\n';
    data.forEach((row) => (output += row.join(delimiter) + '\r\n'));

    const encodedUri = encodeURI(output);
    const link = document.createElement('a');
    link.setAttribute('href', encodedUri);
    link.setAttribute('download', `${filename}.csv`);
    document.body.appendChild(link);

    link.click();
  };
</script>

<div class="overflow-x-auto px-4">
  <table class="table w-full">
    <!-- head -->
    <thead>
      <tr>
        <th>&nbsp;</th>
        {#each headings as heading, i}
          {#if i <= 3}
            <th>
              {heading}
              {#if i < 2}
                <span class="text-red-400">*</span>
              {/if}
            </th>
          {:else}
            <th>
              <input
                placeholder="Ange kolumnnamn..."
                class="bg-transparent p-2 w-full"
                on:keyup={storeData}
                bind:value={heading}
              />
            </th>
          {/if}
        {/each}
        <th class="text-right">
          <button class="btn btn-sm" on:click={addColumn}>Lägg till kolumn</button>
        </th>
      </tr>
    </thead>

    <!-- body -->
    <tbody>
      {#each data as row, i}
        <tr class="m-0 p-0">
          <td>{i + 1}</td>
          {#each row as value}
            <td class="m-0 p-0">
              <input
                placeholder="..."
                class="bg-transparent p-2 w-full"
                on:keyup={storeData}
                bind:value
              />
            </td>
          {/each}
          <td class="m-0 p-0">
            <div class="grid grid-cols-2 gap-4">
              <button class="btn btn-sm" on:click={() => addRow(i)}>Lägg till rad</button>
              {#if data.length > 1}
                <button class="btn btn-sm" on:click={() => removeRow(i)}>Ta bort</button>
              {/if}
            </div>
          </td>
        </tr>
      {/each}
    </tbody>
  </table>
</div>

<div class="text-center">
  <div class="mt-8 ml-2 grid grid-cols-2 gap-4 w-1/4">
    <button class="btn btn-primary" on:click={resetTable}> Återställ tabell </button>
    <button disabled={!isValid} class="btn btn-secondary" on:click={exportToExcel}>
      Exportera till Excel
    </button>
  </div>
</div>
