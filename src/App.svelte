<script lang="ts">
  import { tweened } from "svelte/motion";
  import boardImg from "./assets/Board.png";
  import { cubicOut } from "svelte/easing";
  import { onMount } from "svelte";
  import { fade, scale } from "svelte/transition";

  let players = [{ pos: tweened(1) }];
  let playerPositions: number[] = [];

  onMount(() => {
    players.forEach((player, pi) => {
      player.pos.subscribe((pos) => {
        playerPositions[pi] = pos;
      });
    });
  });

  let twistTable = {
    4: { type: "ladder", pos: 56 },
    12: { type: "ladder", pos: 50 },
    14: { type: "ladder", pos: 55 },
    28: { type: "snake", pos: 10 },
    37: { type: "snake", pos: 3 },
    41: { type: "ladder", pos: 79 },
    48: { type: "snake", pos: 16 },
    54: { type: "ladder", pos: 88 },
    75: { type: "snake", pos: 32 },
    94: { type: "snake", pos: 71 },
    96: { type: "snake", pos: 42 },
  };
  const handleRollRandom = async () => {
    let advance = Math.round(Math.random() * 5 + 1);
    let newPos;
    await players[0].pos.update(
      (pos) => {
        pos = Math.min(pos + advance, 100);
        newPos = pos;
        return pos;
      },
      { duration: advance * 200, easing: cubicOut }
    );

    if (twistTable[newPos]) {
      await players[0].pos.update(
        (pos) => {
          pos = twistTable[pos].pos;

          return pos;
        },
        {
          duration: 10,
          easing: cubicOut,
        }
      );
    }

    // players[0].pos.set(Math.min(players[0].pos + advance, 100), {
    //   duration: 500,
    // });
  };
</script>

<main
  class="flex flex-col items-center justify-center h-screen w-screen bg-gray-900"
>
  <div class="bg-black/50 mb-5 px-2 rounded-md">
    <p class="text-3xl font-bold text-white">
      Aadhi's Electronic Snake & Ladders
    </p>
    <p class="text-gray-500">
      <b class="text-green-400">successfully maintaining</b> websocket connection
      to ESP32 via mDNS
    </p>
  </div>
  <div class="flex flex-row space-x-2 items-center justify-center">
    <div
      class="flex flex-col-reverse justify-center w-min bg-center bg-contain rounded-lg"
      style="background-image: url({boardImg});"
    >
      {#each new Array(10) as _, ri}
        <div
          class="flex items-center"
          style="flex-direction: {ri % 2 == 0 ? 'row' : 'row-reverse'};"
        >
          {#each new Array(10) as _, ci}
            <div
              class="relative ring-3 ring-black/50 rounded-md text-center place-content-center w-[40px] aspect-square"
            >
              <!-- style={twistTable[ri * 10 + ci + 1]
              ? "background-color:rgba(200, 0, 0, 0.4);"
              : ""} -->
              {#each playerPositions as pos}
                {#if Math.round(pos) == ri * 10 + ci + 1}
                  <div
                    out:fade={{ duration: 200 }}
                    class="absolute bg-gradient-to-r ring-[5px] ring-purple-500 from-red-900 to-blue-900 text-white w-full aspect-square rounded-full top-0 text-center place-content-center font-bold text-4xl animate-spin"
                  >
                    {ri * 10 + ci + 1}
                  </div>
                {/if}
              {/each}
            </div>
          {/each}
        </div>
      {/each}
    </div>
    <div class="bg-gray-800 text-white flex flex-col">
      <label for="mdns-url">ESP32 STA mDNS</label><input
        type="text"
        name="mdns-url"
      />
      <label for="battery-percent">Battery Level</label><input
        type="text"
        name="battery-percent"
        value="87.5%"
      />
    </div>
  </div>

  <button
    class="bg-green-500 text-gray-900 p-2 rounded-xl m-2 font-bold text-5xl"
    on:click={handleRollRandom}>random</button
  >
</main>

<style>
  input {
    @apply text-white bg-black text-right;
  }
</style>
