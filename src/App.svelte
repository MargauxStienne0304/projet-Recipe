<script>
  // imports
  import Markdown from "svelte-exmarkdown";
  import Icon from "@iconify/svelte";

  // la liste des recettes
  let recipes = $state([]);

  // plutot que de faire 3 variables séparées, on peut regrouper les valeurs d'une nouvelle recette dans un seul objet
  let newRecipe = $state({
    title: "",
    image: "",
    description: ""
  });

  const baseUrl = "http://127.0.0.1:8090/api/collections/recipes/records";

  // function de gestion de la soumission du formulaire
  async function handleSubmitNewRecipe(event) {
    event.preventDefault();

    const handleRecipe = await createNewRecipe(newRecipe);

    // on ajoute à nos données locales la recette nouvellement créee par l'api (donc avec l'id gérée par l'api)
    recipes.push(handleRecipe);

    // pour reset le formulaire on va REASSIGNER un nouvel objet à newRecipe
    // ce qui a pour effet de rendre unique la recette dans le tableau
    newRecipe = {
      title: "",
      image: "",
      description: ""
    };
  }

  // function de gestion de la suppression
  async function handleDelete(idToDelete) {
    console.log("tu essaies de supprimer la recette", idToDelete);

    // supprimer la recette de l'API
    await handleDeleteRecipe(idToDelete);

    // pour supprimer une recette de la liste des recettes "recipes"
    // on va construire un nouveau tableau qui contiendra toutes les recettes SAUF celle à supprimer
    recipes = recipes.filter((recipe) => recipe.id !== idToDelete);
  }


  // ----
  // API
  // ----
  // fonction qui recupere les recettes depuis l'api
  async function getRecipes() {
    const response = await fetch(baseUrl);
    const recipesApiData = await response.json();
    return recipesApiData.items;
  }

  // function qui supprime une recette dans l'api
  async function handleDeleteRecipe(id) {
    const response = await fetch(`${baseUrl}/${id}`, {
      method: "DELETE",
    });
  recipes = recipes.filter((recipe) => recipe.id !== id);
  }

  // function qui crée une nouvelle recette dans l'api

  async function createNewRecipe(newRecipe){
    const response = await fetch(baseUrl, {
      method: "POST",
      headers: {"content-type": "application/json"},
      body: JSON.stringify(newRecipe)
    });

    const data = await response.json();
    return data;    
  }


  // au moment du chargement du composant
  $effect(async () => {
    recipes = await getRecipes();
  });
  

</script>

<div class="container">

  <header>
    <h1>
      <a href="/">
        <img src="logo.png" alt="O'Recipes">
      </a>
    </h1>
  </header>

  <main>

    <section>
      <h2>Ajouter une recette</h2>

      <form class="form" onsubmit={handleSubmitNewRecipe}>

        <div class="form__group">
            <label class="form__label" for="title">Titre</label>
            <input class="form__input" type="text" id="title" bind:value={newRecipe.title}>
        </div>

        <div class="form__group">
            <label class="form__label" for="image">Image URL</label>
            <input class="form__input" type="text" id="image" bind:value={newRecipe.image}>
        </div>

        <div class="form__group">
            <label class="form__label" for="description">Description</label>
            <textarea class="form__input form__input--textarea" id="description" bind:value={newRecipe.description}></textarea>
        </div>

        <button class="form__button" type="submit">Ajouter</button>
      
      </form>

    </section>

    <section>
      <h2>Les {recipes.length} recettes</h2>

      <div class="recipes">

        {#each recipes as recipe}  
        <!-- une recette -->
        <article class="recipe" data-id={recipe.id}>

          <div class="recipe__actions">
            <button
              class="recipe__button"
              aria-label="Modifier la recette {recipe.title}"
            >
              <Icon icon="typcn:pencil" width="20" height="20" color="#FFF" />
            </button>

            <button
              class="recipe__button"
              aria-label="Supprimer la recette {recipe.title}"
              onclick={() => handleDelete(recipe.id)}
            >
              <Icon icon="typcn:times-outline" width="20" height="20" color="#FFF" />
            </button>
            
          </div>

          <img class="recipe__image" src={recipe.image} alt="">
          <h2 class="recipe__title">{recipe.title}</h2>
          <Markdown md={recipe.description} />

          <!-- debug 
          <p>id : {recipe.id}</p>-->

          <h2>Modifier une recette</h2>
          <form>

            <div>
              <label for="change-title">Titre</label>
              <input type="text" id="change-title">
            </div>

            <div>
              <label for="change-image"> Image</label>
              <input type="text" id="change-image">
            </div>

            <div>
              <label for="change-description">Description</label>
              <textarea type="text" id="change-description"></textarea>
            </div>

            <button type="button">Annuler</button>
            <button type="submit">Modifier</button>
          </form>


        </article>
        {/each}

      </div>
    </section>

  </main>


</div>

<style>
  img {
    height: 100px;
  }

  .container {
    max-width: 1000px;
    margin: 0 auto;
  }

  .recipes {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 2rem;
  }

  .recipe {
    padding: 1rem;
    border-radius: .4rem;
    box-shadow: 0 0 .5rem rgb(0 0 0 / 20%);
    position: relative;
  }

  .recipe__image {
    height: 100px;
    width: 100%;
    border-radius: .25rem;
    object-fit: cover;
  }

  .recipe__actions {
    position: absolute;
    top: -1rem;
    right: -1rem;
    display: flex;
    gap: .25rem;
  }

  .recipe__button {
    background: chocolate;
    border: none;
    border-radius: 50%;
    width: 2rem;
    aspect-ratio: 1 / 1;
    cursor: pointer;

    /* centrage de l'icone */
    display: flex;
    justify-content: center;
    align-items: center;
  }


  .form {
    display: flex;
    flex-direction: column;
    gap: 1rem;
  }

  .form__group {
    display: flex;
    flex-direction: column;
  }

  .form__input {
    border: solid 1px rgb(0 0 0 / 20%);
    border-radius: .25rem;
    padding: .5rem;
    font-family: inherit;
  }

  .form__input--textarea {
    height: 5rem;
    resize: vertical;
  }

  .form__button {
    align-self: flex-start;
    padding: .5rem 1rem;
    border: none;
    background: chocolate;
    color: white;
    border-radius: .25rem;
    cursor: pointer;
  }


</style>
