<script>
  import "../[id]/recept.scss";
  import { CornerDownLeft } from "lucide-svelte";
  import { Users } from "lucide-svelte";
  import { page } from "$app/stores";
  import { derived } from "svelte/store";

  const lastSegment = derived(page, ($page) => {
    const segments = $page.url.pathname.split("/");
    return segments[segments.length - 1];
  });

  let segment = $state("");
  let r_data = $state([]);
  $effect(() => {
    segment = $lastSegment;
  });
  let is_toggled = $state(false);
  let wakeLock = null;
  let fetch_recipe = () => {
    fetch(`https://676c34c20e299dd2ddfc4354.mockapi.io/recipes/${segment}`)
      .then((response) => {
        return response.json();
      })
      .then((data) => {
        console.log(data);
        r_data = data;
      });
  };
  async function requestWakeLock() {
    try {
      if (!wakeLock) {
        wakeLock = await navigator.wakeLock.request("screen");
        console.log("Wake Lock aktiviran!");
      }
    } catch (err) {
      console.error("Greška pri aktivaciji Wake Lock-a:", err);
    }
  }

  async function releaseWakeLock() {
    if (wakeLock) {
      await wakeLock.release();
      wakeLock = null;
      console.log("Wake Lock deaktiviran!");
    }
  }

  $effect(() => {
    if (is_toggled) {
      requestWakeLock();
    } else {
      releaseWakeLock();
    }
  });
  $effect(() => {
    fetch_recipe();
  });
</script>

<a href="/" class="redirect">
  <CornerDownLeft />
  lista recepata</a
>

<div class="container">
  <h1>{r_data.title}</h1>

  <p class="users"><Users />{r_data.persons}</p>

  <div class="switch_component">
    <p>ne gasi ekran</p>
    <label class="switch">
      <input
        type="checkbox"
        onchange={(e) => (is_toggled = e.target.checked)}
      />
      <span class="slider round"></span>
    </label>
  </div>
  <div class="table_component" role="region">
    <table>
      <thead>
        <tr>
          <th>sastojci</th>
          <th>količina</th>
        </tr>
      </thead>
      <tbody>
        {#if r_data.ingredients}
          {#each Object.entries(r_data.ingredients) as [key, value]}
            <tr>
              <td>{value.key}</td>
              <td>{value.value}</td>
            </tr>
          {/each}
        {/if}
      </tbody>
    </table>
  </div>
  <div class="prep">
    <p>priprema:</p>
    <p>{r_data.prep}</p>
  </div>
</div>
