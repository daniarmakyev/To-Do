<script lang="ts">
  import { api } from "../lib/constants/constants";
  import { createMutation } from "@tanstack/svelte-query";
  import Input from "$lib/Input.svelte";
  import type { Post } from "../lib/constants/types";
  import type { AxiosError, AxiosResponse } from "axios";

  let post: Post = {
    title: "",
    desc: " ",
    completed: false,
  };

  let error: string = "";

  const addMutation = createMutation({
    mutationFn: async (post: Post): Promise<AxiosResponse> => {
      return api.post("posts", post);
    },
    onSuccess: () => {
      console.log("Task added");
      post.title = "";
    },
    onError: (err: AxiosError) => {
      error =
        `Error: ${err.response?.status}, ${err.response?.data}` ||
        err.code ||
        "Unexpected error";
    },
  });

  const validateAndSubmit = (e: Event) => {
    e.preventDefault();
    e.stopPropagation();

    if (!post.title.trim()) {
      error = "The field must be filled!";
      return;
    }

    error = "";

    $addMutation.mutate(post);
  };
</script>

<main class="max-w-4xl mx-auto">
  <h1 class="text-3xl mt-2 mx-auto max-w-36 font-semibold">To Do List</h1>
  <form
    class="mt-5 flex items-center justify-center gap-2"
    on:submit={validateAndSubmit}
  >
    <section class="flex flex-col items-center relative pb-7">
      <Input bind:value={post.title} placeholder="To Do" />
      {#if error}
        <span class="mt-2 text-red-500 absolute bottom-0 left-7 w-64 text-sm"
          >{error}</span
        >
      {/if}
    </section>
    <button class="p-2 border-[1px] border-gray-950 rounded-md mt-3 mb-4">
      Add
    </button>
  </form>
</main>
