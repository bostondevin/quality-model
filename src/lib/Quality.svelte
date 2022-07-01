<script lang="ts">
  import { onMount } from "svelte";
  import { writable } from "svelte/store";
  import { browser } from "$app/env";

  import { QualityData } from "./quality-data";
  import Rating from "./Rating.svelte";

  const dataStore = writable(null);
  const totalsStore = writable(null);

  function toLetterGrade(value: number) {
    let textG = "";
    if (value >= 100) {
      textG = "A+";
    } else if (value >= 85) {
      textG = "A";
    } else if (value >= 80) {
      textG = "A-";
    } else if (value >= 75) {
      textG = "B+";
    } else if (value >= 70) {
      textG = "B";
    } else if (value >= 65) {
      textG = "B-";
    } else if (value >= 60) {
      textG = "C+";
    } else if (value >= 55) {
      textG = "C";
    } else if (value >= 50) {
      textG = "D";
    } else {
      textG = "F";
    }
    return textG;
  }

  function dynamicSort(property) {
    var sortOrder = 1;

    if (property[0] === "-") {
      sortOrder = -1;
      property = property.substr(1);
    }

    return function (a, b) {
      if (sortOrder == -1) {
        return b[property].localeCompare(a[property]);
      } else {
        return a[property].localeCompare(b[property]);
      }
    };
  }

  const comparables = [
    {
      id: 1,
      name: "React",
    },
    {
      id: 2,
      name: "SvelteKit",
    },
    {
      id: 3,
      name: "Serverless",
    },
    {
      id: 4,
      name: "Server",
    },
  ];

  const changeStar = (e, item, comparable) => {
    const d = e.detail;
    const newVal = JSON.parse($dataStore);
    newVal[item.parentId][item.id][comparable.id] = d.value;
    dataStore.set(JSON.stringify(newVal));
  };

  let cumulativeResults;
  $: cumulativeResults;

  let cumulativeTotals;
  $: cumulativeTotals;

  let overallTotals = [];
  $: overallTotals;

  totalsStore.subscribe((d) => {
    if (d) {
      cumulativeTotals = JSON.parse(d);

      overallTotals = [];

      comparables.forEach((comparable) => {
        overallTotals.push({ id: comparable.id, total: 0, count: 0 });
      });

      cumulativeTotals.forEach((cumTot) => {
        Object.keys(cumTot.results).forEach((key, i) => {
          overallTotals[i].count =
            overallTotals[i].count + cumTot.results[key].count;
          overallTotals[i].total =
            overallTotals[i].total + cumTot.results[key].total;
        });
      });
    } else {
      cumulativeTotals = null;
    }
  });

  dataStore.subscribe((d) => {
    cumulativeResults = JSON.parse(d);

    if (cumulativeResults) {
      isResettable = true;
      localStorage.setItem("cumulativeResults", d);
      let totals = [];

      Object.keys(cumulativeResults).forEach((parent) => {
        let hasValue = false;

        let newObj = {
          id: parseInt(parent),
          results: {},
        };

        comparables.forEach((comparable) => {
          newObj.results[comparable.id] = { count: 0, total: 0, uncounted: 0 };
        });

        Object.keys(cumulativeResults[parent]).forEach((attribute) => {
          Object.keys(cumulativeResults[parent][attribute]).forEach(
            (comparable) => {
              if (cumulativeResults[parent][attribute][comparable] !== 0) {
                hasValue = true;
                newObj.results[comparable].count++;
                newObj.results[comparable].total =
                  newObj.results[comparable].total +
                  cumulativeResults[parent][attribute][comparable];
              } else {
                newObj.results[comparable].uncounted++;
              }
            }
          );
        });

        if (hasValue) {
          totals.push(newObj); // { id: parent, total: ((cumTotal / numItems) * 100) / 5 }
        }
      });

      totalsStore.set(JSON.stringify(totals));
    }
  });

  let loaded = false;
  let isResettable = false;

  const clear = () => {
    localStorage.setItem("cumulativeResults", "");
    dataStore.set(null);
    totalsStore.set(null);
    resetDataStore();
    isResettable = false;
  };

  const resetDataStore = () => {
    let results = {};

    QualityData.filter((d) => d.parentId === 0).forEach((parent) => {
      results[parent.id] = {};

      QualityData.filter((d) => d.parentId === parent.id).forEach((d) => {
        results[parent.id][d.id] = {};

        comparables.forEach((comparable) => {
          results[parent.id][d.id][comparable.id] = 0;
        });
      });
    });

    dataStore.set(JSON.stringify(results));
  };

  onMount(() => {
    const existingData = localStorage.getItem("cumulativeResults");
    if (existingData) {
      isResettable = true;
      dataStore.set(existingData);
    } else {
      resetDataStore();
    }

    loaded = true;
  });
</script>

{#if isResettable}
  <button on:click={clear}>Clear Results</button>
{/if}
<div class="flex w-full flex-col h-full text-white">
  {#if overallTotals.filter((d) => d.count > 0).length > 0}
    <div class="grid grid-cols-6 gap-4 place-content-center bg-slate-800 p-4">
      {#each overallTotals.filter((d) => d.count > 0) as item (item.id)}
        <div class="text-center">
          <div class="text-3xl">
            {Math.round(((item.total / item.count) * 100) / 5)}%
          </div>

          <h2 class="mb-1 font-xl bold uppercase center">
            {comparables.find((d) => d.id == item.id)?.name}
          </h2>

          <div
            class="my-1 rounded bg-slate-400"
            style="min-width: 120px; height: 22px;"
          >
            <div
              class="h-full rounded bg-blue-500"
              style="width: {Math.round(
                ((item.total / item.count) * 100) / 5
              )}%"
            />
          </div>
        </div>
      {/each}
    </div>
  {/if}

  {#if cumulativeTotals}
    <div class="flex bg-slate-800 text-white p-4">
      <div class="grid grid-cols-6 gap-4">
        {#each cumulativeTotals as item (item.id)}
          <div class="bg-slate-700 p-3 rounded-lg">
            <h2 class="mb-1 text-lg uppercase text-center">
              {QualityData.find((d) => d.id == item.id)?.heuristic}
            </h2>

            <table class="text-sm w-full">
              <tbody>
                {#each comparables as comparable (comparable.id)}
                  <tr>
                    <th class="text-left pr-2">{comparable.name}</th>
                    <td class="flex" style="width: 90px;">
                      {#if item.results[comparable.id].count > 0}
                        <div
                          class="my-1 rounded bg-slate-400 w-full"
                          style="height: 22px;"
                        >
                          <div
                            class="h-full rounded bg-blue-500"
                            style="width: {((item.results[comparable.id].total /
                              item.results[comparable.id].count) *
                              100) /
                              5}%"
                          />
                        </div>
                      {:else}
                        <div
                          style="font-size: 12px; opacity: .5; margin: 5px 0"
                        >
                          {item.results[comparable.id].uncounted} Unrated
                        </div>
                      {/if}
                    </td>
                  </tr>
                {/each}
              </tbody>
            </table>

            <div class="pt-2 text-xs opacity-75 mt-2">
              {QualityData.find((d) => d.id == item.id)?.description}
            </div>
          </div>
        {/each}
      </div>
    </div>
  {/if}

  <div class="flex bg-slate-600 grow p-4">
    {#if loaded}
      <div class="grow overflow-y-auto grid grid-cols-6 gap-4">
        {#each QualityData.filter((d) => d.parentId !== 0).sort(dynamicSort("heuristic")) as item (item.id)}
          <div class="bg-slate-500 hover:bg-slate-700 p-3 rounded-lg">
            <h2 class="mb-1 text-lg uppercase text-center">
              {item.heuristic}
            </h2>

            <div class="center text-xs opacity-50 mb-2 text-center">
              {QualityData.find((d) => d.id === item.parentId)?.heuristic}
            </div>

            <table class="text-sm w-full">
              <tbody>
                {#each comparables as comparable (comparable.id)}
                  <Rating
                    {comparable}
                    {item}
                    value={cumulativeResults[item.parentId][item.id][
                      comparable.id
                    ]}
                    on:change={(e) => changeStar(e, item, comparable)}
                  />
                {/each}
              </tbody>
            </table>

            <div class="pt-2 text-xs opacity-75 mt-2">
              {item.description}
            </div>
          </div>
        {/each}
      </div>
    {/if}
  </div>
</div>
