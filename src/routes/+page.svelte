<script>
  import "../routes/index.scss";
  import { Search } from "lucide-svelte";
  import { CirclePlus } from "lucide-svelte";
  import { X } from "lucide-svelte";
  import { Package } from "lucide-svelte";
  import { writable } from "svelte/store";
  let api = "https://676c34c20e299dd2ddfc4354.mockapi.io/recipes/";

  let recipe = $state([]);
  let searchQuery = writable("");
  let filteredRecipes = $state([]);
  let title = $state("");
  let prep = $state("");
  let persons = $state("");
  let ingredient_array = $state([]);
  let ingredientsUI = $state([]); // Služi za upravljanje prikazom dodatih inputa
  let ingredientKey = ""; // Privremeni unos za ključ
  let ingredientValue = ""; // Privremeni unos za vrednost

  $effect(() => {
    filteredRecipes = recipe.filter((r) =>
      r.title.toLowerCase().includes($searchQuery.toLowerCase())
    );
  });

  let fetch_recipes = () => {
    fetch(api)
      .then((response) => response.json())
      .then((data) => {
        console.log(data);
        recipe = data;
      });
  };

  let post_recipe = () => {
    fetch(api, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        title: title,
        prep: prep,
        persons: persons,
        ingredients: ingredient_array,
      }),
    })
      .then((response) => response.json())
      .then((data) => {
        console.log(data);
        setTimeout(() => {
          window.location.reload();
        }, 2000);
      });
  };

  fetch_recipes();

  const openDialog = () => {
    const dialog = document.querySelector(".alert_dialog_component");
    dialog.showModal();
    dialog.addEventListener("click", (e) => {
      if (e.target === dialog) {
        closeDialog();
      }
    });
    dialog.style.animation = "fadeInScale 0.3s ease forwards";
  };

  const closeDialog = () => {
    const dialog = document.querySelector(".alert_dialog_component");
    dialog.close();
  };

  const addIngredient = () => {
    ingredientsUI = [...ingredientsUI, { key: "", value: "" }];
  };

  const removeIngredient = (index) => {
    ingredientsUI = ingredientsUI.filter((_, i) => i !== index);
    ingredient_array = ingredient_array.filter((_, i) => i !== index);
  };

  const updateIngredient = (index, key, value) => {
    if (ingredient_array[index]) {
      ingredient_array[index] = { key, value };
    } else {
      ingredient_array = [...ingredient_array, { key, value }];
    }
  };
</script>

<div class="search_wrapper">
  <input type="text" placeholder="pretrazi recept" bind:value={$searchQuery} />
  <Search size={32} />
</div>

<button class="post_recipe" onclick={openDialog}>
  <CirclePlus /> objavi recept</button
>
<dialog class="alert_dialog_component">
  <div class="alert_dialog_content">
    <p>objavi recept</p>
    <div class="mini_wrapper">
      <p>naslov</p>
      <input type="text" placeholder="unesi naslov" bind:value={title} />
    </div>
    <div class="mini_wrapper">
      <p>osobe</p>
      <input type="number" placeholder="za koliko osoba" bind:value={persons} />
    </div>
    <div class="mini_wrapper">
      <p>priprema</p>
      <textarea placeholder="kako se ovo jelo priprema" bind:value={prep}
      ></textarea>
    </div>
    {#each ingredientsUI as ingredient, index}
      <div class="double_input_wrapper">
        <input
          type="text"
          placeholder="sastojak"
          bind:value={ingredient.key}
          oninput={(e) =>
            updateIngredient(index, e.target.value, ingredient.value)}
        />
        <input
          type="text"
          placeholder="kolicina"
          bind:value={ingredient.value}
          oninput={(e) =>
            updateIngredient(index, ingredient.key, e.target.value)}
        />
        <X onclick={() => removeIngredient(index)} />
      </div>
    {/each}
    <button onclick={addIngredient}>
      <CirclePlus /> dodaj sastojak
    </button>
    <button class="close" onclick={closeDialog}>
      <X /> izadji iz objavljivanja</button
    >
    <button class="post" onclick={post_recipe}>
      <Package /> objavi recept</button
    >
  </div>
</dialog>
<div class="recipes_wrapper">
  {#each filteredRecipes as recipe}
    <a href="/recept/{recipe.id}">{recipe.title}</a>
  {/each}
</div>
