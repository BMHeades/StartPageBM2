<script lang="ts">
  // localStorage.clear();

  const defaultWelcomeMessage = "~$hello!";
  const defaultCategories = [
    {
      name: "Dev",
      items: [
        {
          name: "github",
          url: "https://github.com/BMHeades/StartPageBM2",
        },
        {
          name: "boot.dev",
          url: "https://www.boot.dev?bannerlord=bmheades",
        },
      ],
    },
    {
      name: "Study",
      items: [
        {
          name: "canvas",
          url: "https://instructure.com/",
        },
        {
          name: "ctclink",
          url: "https://csprd.ctclink.us",
        },
        {
          name: "wamap",
          url: "https://www.wamap.org/",
        },
      ],
    },
    {
      name: "Other",
      items: [
        {
          name: "youtube",
          url: "https://www.youtube.com/",
        },
      ],
    },
  ];
  // clean start
  if (
    !localStorage.getItem("welcomeMessage") ||
    localStorage.getItem("welcomeMessage") === ""
  ) {
    localStorage.setItem("welcomeMessage", defaultWelcomeMessage);
  }
  if (
    !localStorage.getItem("categories") ||
    localStorage.getItem("categories") === ""
  ) {
    localStorage.setItem("categories", JSON.stringify(defaultCategories));
  }

  type Link = {
    name: string;
    url: string;
  };
  type Category = {
    name: string;
    items: Link[];
  };

  let categories: Category[] = $state(
    JSON.parse(localStorage.getItem("categories") as string),
  );

  let welcomeMessage: string = $state(
    localStorage.getItem("welcomeMessage") as string,
  );

  let linkCount = $derived(
    categories.reduce((total, current) => {
      return total + current.items.length;
    }, 0),
  );

  $effect(() => {
    localStorage.setItem("welcomeMessage", welcomeMessage);
    localStorage.setItem("categories", JSON.stringify(categories));
  });

  let isEditMode = $state(false);
  function toggleEditMode() {
    isEditMode = !isEditMode;
  }

  function addLink(category: Category) {
    category.items.push({
      name: "link" + (linkCount + 1),
      url: "https://example.com/",
    });
  }
  function removeLink(category: Category, link: Link) {
    category.items = category.items.filter((item) => item.name !== link.name);
  }

  function addCategory() {
    categories.push({
      name: "Other" + (categories.length + 1),
      items: [
        {
          name: "link" + linkCount,
          url: "https://example.com/",
        },
      ],
    });
  }
  function deleteCategory(category: Category) {
    categories = categories.filter((item) => item.name !== category.name);
  }

  function exit(event: any) {
    // console.log('Key released:', event.key);
    if(event.key === "Escape" || event.key === "Enter"){
      toggleEditMode()
    }
  }

  function exportLocalStorage() {
    const data: Record<string, string> = {};
    for (let i = 0; i < localStorage.length; i++) {
      const key = localStorage.key(i);
      if (key) {
        data[key] = localStorage.getItem(key) || "";
      }
    }
    const blob = new Blob([JSON.stringify(data, null, 2)], { type: "application/json" });
    const url = URL.createObjectURL(blob);
    const a = document.createElement("a");
    a.href = url;
    a.download = "startpage-config.json";
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    URL.revokeObjectURL(url);
  }

  let fileInput: HTMLInputElement = $state()!;

  function importLocalStorage(event: Event) {
    const input = event.target as HTMLInputElement;
    if (!input.files || input.files.length === 0) return;
    const file = input.files[0];
    const reader = new FileReader();
    reader.onload = (e) => {
      try {
        const text = e.target?.result as string;
        const data = JSON.parse(text);
        
        // Clear current localStorage and set new values
        localStorage.clear();
        for (const key in data) {
          localStorage.setItem(key, data[key]);
        }
        
        // Ensure defaults if keys are missing/empty
        if (!localStorage.getItem("welcomeMessage")) {
          localStorage.setItem("welcomeMessage", defaultWelcomeMessage);
        }
        if (!localStorage.getItem("categories")) {
          localStorage.setItem("categories", JSON.stringify(defaultCategories));
        }

        // Update reactive state
        welcomeMessage = localStorage.getItem("welcomeMessage") as string;
        categories = JSON.parse(localStorage.getItem("categories") as string);
        
        // Reset file input value
        input.value = "";
      } catch (err) {
        alert("Failed to parse the imported JSON file. Please ensure it is a valid configuration.");
        console.error(err);
      }
    };
    reader.readAsText(file);
  }
</script>

<!-- Welcome Message -->
<div class="h-50 md:h-60 lg:h-70 flex justify-center items-center text-6xl md:text-8xl lg:text-9xl font-extrabold text-title pt-15 mb-10">
    {#if isEditMode}
      <input class="w-150 bg-input-bg  rounded-2xl" bind:value={welcomeMessage} onkeyup={exit}/>
    {:else}
      <h1>{welcomeMessage}</h1>
    {/if}

</div>

<!-- <div class="grid grid-cols-2 gap-6 md:grid-cols-3 lg:grid-cols-4 p-10 px-20 md:px-25 lg:px-50"> -->
<div class="lg:mx-[8%] grid grid-cols-[repeat(auto-fit,minmax(250px,1fr))]">


  <!-- Categories -->
  {#each categories as category, i}

    <div class="min-h-25 min-w-25 max-w-100 mx-auto">
      
      <div class={isEditMode? "mx-auto w-1/1" : "mx-auto w-1/2"}>
        <!-- Category Title -->
      <div class="flex items-center h-15 text-2xl font-bold text-category">
        {#if isEditMode}
          ~/<input class="min-w-0 max-w-2/5 bg-input-bg" bind:value={category.name} onkeyup={exit} />
          <button class="btn-delete" onclick={() => deleteCategory(category)}>Delete Category</button>
        {:else}
          <h2 class="text-1.5xl">~/{category.name}</h2>
        {/if}
      </div>

      <!-- Links -->
      <ul>
        {#each category.items as link}
          <li class="flex gap-2 h-7 w-75">
            {#if isEditMode}
              <input class="text-xs w-3/10 bg-input-bg" bind:value={link.name} onkeyup={exit}/><input class="text-xs w-5/10 bg-input-bg" bind:value={link.url} onkeyup={exit} /><button class="btn-delete w-2/10 py-0.5" onclick={() => removeLink(category, link)}>Delete</button>
            {:else}
              <a href={link.url}>{link.name}</a>
            {/if}
          </li>
        {/each}

        <!-- Add Link Button -->
        {#if isEditMode}
          <li>
            <button class="btn-add py-0.5 ml-2" onclick={() => addLink(category)}>Add Link</button>
          </li>
        {/if}
      </ul>
      </div>
      
    </div>
  {/each}

  
  
  
</div>
<!-- Add Category / Export / Import Config Buttons -->
{#if isEditMode}
  <div class="fixed bottom-4 left-4 flex gap-2">
    <button class="btn-add" onclick={addCategory}>Add Category</button>
    <button class="btn-export" onclick={exportLocalStorage}>Export Config</button>
    <button class="btn-import" onclick={() => fileInput.click()}>Import Config</button>
    <input
      type="file"
      accept=".json"
      bind:this={fileInput}
      style="display: none;"
      onchange={importLocalStorage}
    />
  </div>
{/if}

<!-- Edit Mode Button -->
<button class="opacity-0 hover:opacity-100 h-1/10 w-1/10 fixed bottom-4 right-4 {isEditMode? "btn-add opacity-100": "btn-delete opacity-0 hover:opacity-100"}" onclick={toggleEditMode}>Edit Mode: {isEditMode? "ON": "OFF"}</button>
