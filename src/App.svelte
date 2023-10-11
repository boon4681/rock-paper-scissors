<script lang="ts">
  import "carbon-components-svelte/css/g90.css";
  import { Button, Slider } from "carbon-components-svelte";
  import { onMount } from "svelte";
  const limit = (min: number, max: number, a: number) => {
    return Math.max(min, Math.min(max, a));
  };
  const sc_limit = (a: number) => {
    return Math.max(-0.1, Math.min(0.1, a));
  };
  const speed = () => {
    return sc_limit(Math.random() - 0.5);
  };
  const tweek_speed = (m: any) => {
    m.x += speed() / 10;
    m.y += speed() / 10;
    return m;
  };
  const create = (id: number) => {
    return {
      id,
      x: Math.random(),
      y: Math.random(),
      type: ["📜", "✂️", "🪨"][Math.floor(Math.random() * 3)],
      attacked: false,
      v: { x: speed(), y: speed() },
    };
  };
  let value = 30;
  let entities = new Array(value).fill("").map((_, i) => create(i));

  let k_constant = Math.sqrt(
    Math.pow(30 / window.innerWidth, 2) + Math.pow(30 / window.innerHeight, 2)
  );
  const distant_from_to = (a: any, b: any) => {
    return [Math.sqrt(Math.pow(a.x - b.x, 2) + Math.pow(a.y - b.y, 2)), b.id];
  };
  const win_table = {
    "📜": "🪨",
    "🪨": "✂️",
    "✂️": "📜",
  } as any;
  const is_a_win_b = (a: any, b: any) => {
    return win_table[a.type] == b.type;
  };
  const collition_all = (a: any) => {
    const all = entities
      .filter((n) => n.id != a.id)
      .map((n) => distant_from_to(a, n));
    let lose_least;
    for (const n of all) {
      if (
        !lose_least &&
        is_a_win_b(a, entities[n[1]]) &&
        n[0] < k_constant * 10
      ) {
        lose_least = n;
      }
      if (lose_least && n[0] < lose_least[0] && is_a_win_b(a, entities[n[1]])) {
        lose_least = n;
      }
    }
    if (lose_least) {
      let target = entities[lose_least[1]];
      target.v = tweek_speed(Object.assign({}, a.v as any));
      target.attacked = true;
      if (lose_least && lose_least[0] < k_constant * 0.5) {
        target.type = a.type;
      }
    }
  };
  let fps = 0;
  onMount(() => {
    let win: number;
    let last = Date.now();
    let delta = 0;
    const render = () => {
      delta = (Date.now() - last) / 1000;
      last = Date.now();
      fps = Math.floor(1 / delta);
      for (const e of entities) {
        e.x += e.v.x / 100;
        e.y += e.v.y / 100;
        if (e.x >= 1 || e.x <= 0) e.v.x = -e.v.x;
        if (e.y >= 1 || e.y <= 0) e.v.y = -e.v.y;
        e.v.x = sc_limit(e.v.x);
        e.v.y = sc_limit(e.v.y);
        e.x = limit(0, 1, e.x);
        e.y = limit(0, 1, e.y);
        if (!e.attacked) {
          collition_all(e);
        }
      }
      for (const e of entities) {
        e.attacked = false;
      }
      entities = entities;
      win = window.requestAnimationFrame(render);
    };
    win = window.requestAnimationFrame(render);
    return () => {
      window.cancelAnimationFrame(win);
    };
  });
</script>

<main>
  {#each entities as entity, _ (entity.id)}
    <div
      class="f"
      style="--x:{entity.x};--y:{entity.y};"
      data-type={entity.type}
    />
  {/each}
  <div class="menu">
    <div>{fps} fps</div>
    <Slider labelText="Elements" bind:value max={500} />
    <div>
      <Button
        on:click={() => {
          entities = new Array(value).fill("").map((_, i) => create(i));
        }}
        size="small">Apply and Reset</Button
      >
    </div>
  </div>
</main>

<style>
  .menu {
    display: flex;
    flex-direction: column;
    background-color: black;
    width: 370px;
    padding: 10px;
  }
  main {
    position: relative;
    width: 100vw;
    height: 100vh;
    font-size: 30px;
    overflow: hidden;
    user-select: none;
  }
  .f {
    position: absolute;
    font-size: 1em;
    width: 1em;
    height: 1em;
    left: calc(var(--x) * 100% - 0.5em);
    top: calc(var(--y) * 100% - 0.5em);
  }
  .f::after {
    position: absolute;
    content: attr(data-type);
  }
</style>
