<script>
  import { fade } from 'svelte/transition';
  import { afterUpdate } from 'svelte';
  const arraySize = 10;
  let cols = [];
  let adjacents = [];
  let targ = null;
  let lastTarg = null;
  let moving = false;
  const moveTransform = ['0, 100%', '0, -100%', '100%, 0', '-100%, 0'];

  class Square {
    constructor(value, x = 0, y = 0) {
      this.value = value;
      this.x = x;
      this.y = y;
      this.move = null;
    }
  }

  for (let i = 0; i < arraySize; i++) {
    cols.push([]);
    for (let j = 0; j < arraySize; j++) {
      cols[i].push(new Square(Math.floor(Math.random() * 9 + 1), i, j));
    }
  }
  $: if (targ) {
    const direction = adjacents.indexOf(targ);
    if (direction != -1) {
      [targ.value, lastTarg.value] = [lastTarg.value, targ.value];
      adjacents = [];
      swap(direction);
      cols = cols;
    } else {
      adjacents = [
        targ.y > 0 ? cols[targ.x][targ.y - 1] : null,
        targ.y + 1 < arraySize ? cols[targ.x][targ.y + 1] : null,
        targ.x > 0 ? cols[targ.x - 1][targ.y] : null,
        targ.x + 1 < arraySize ? cols[targ.x + 1][targ.y] : null,
      ];
      lastTarg = targ;
    }
  }
  function swap(direction) {
    moving = true;
    targ.move = moveTransform[direction];
    lastTarg.move = moveTransform[(direction % 2) * -2 + 1 + direction];
    setTimeout(() => {
      targ.move = null;
      lastTarg.move = null;
      targ = null;
      lastTarg = null;
      cols = cols;
    }, 500);
    setTimeout(() => checkMatches(), 600);
  }
  function checkMatches() {
    let matches = new Set();
    for (let x = 0; x < arraySize; x++) {
      for (let y = 0; y < arraySize; y++) {
        if (x + 2 < arraySize) {
          const lrMatches = [cols[x][y], cols[x + 1][y], cols[x + 2][y]];
          if (lrMatches.every((el) => el.value === lrMatches[0].value)) {
            lrMatches.forEach((match) => matches.add(match));
          }
        }
        if (y + 2 < arraySize) {
          const udMatches = [cols[x][y], cols[x][y + 1], cols[x][y + 2]];
          if (udMatches.every((el) => el.value === udMatches[0].value)) {
            udMatches.forEach((match) => matches.add(match));
          }
        }
      }
    }

    cols = cols.map((col) => col.filter((x) => !matches.has(x)));
    if (matches.size === 0) {
      moving = false;
    } else {
      setTimeout(() => {
        refill();
        moving = false;
        checkMatches();
      }, 500);
    }
  }
  function refill() {
    for (let x = 0; x < arraySize; x++) {
      const shortage = arraySize - cols[x].length;
      for (let i = 0; i < shortage; i++) {
        cols[x].unshift(new Square(Math.floor(Math.random() * 9 + 1), x, 0));
      }
      for (let y = 0; y < arraySize; y++) {
        cols[x][y].y = y;
      }
      cols = cols;
    }
  }
</script>

<main>
  <div id="grid">
    {#each cols as col, i}
      <div class="column">
        {#each col as square, j (square)}
          <!-- svelte-ignore a11y-no-noninteractive-tabindex -->
          <div
            tabindex="0"
            class="square"
            on:click={() => {
              if (!moving) {
                targ = square;
              }
            }}
            on:keydown={(e) => {
              if (e.key === ' ') {
                e.preventDefault();
                if (!moving) {
                  targ = square;
                }
              }
            }}
            class:target={targ === square}
            class:adjacent={adjacents.includes(square)}
            class:move={square.move}
            style="--move-direction: {square.move}"
            out:fade={{ duration: 500 }}>
            {square.value}
          </div>
        {/each}
      </div>
    {/each}
  </div>
</main>

<style>
  #grid {
    display: flex;
    align-items: flex-end;
  }
  .column {
    display: flex;
    flex-flow: column;
  }
  .square {
    border: 2px solid black;
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 1.4em;
    height: 2em;
    width: 2em;
    background-color: white;
    color: black;
    user-select: none;
    cursor: pointer;
  }
  .target {
    border-color: turquoise;
    background-color: turquoise;
  }
  .adjacent {
    border-color: pink;
    background-color: #ff69b4;
  }
  .move {
    animation: moveBack linear 0.5s forwards;
    transform: translate(var(--move-direction));
  }
  @keyframes moveBack {
    to {
      transform: none;
    }
  }
</style>
