<script lang="ts">
  import { api } from "../lib/constants/constants";
  import {
    createMutation,
    createQuery,
    useQueryClient,
  } from "@tanstack/svelte-query";
  import Input from "$lib/Input.svelte";
  import type { Post } from "../lib/constants/types";
  import type { AxiosError, AxiosResponse } from "axios";

//!тут переменнные через  let  как состояни в реакте работают

  let post: Post = {
    title: "",
    desc: " ",
    completed: false,
  };

  let task: Post = {
    title: "",
    desc: " ",
    completed: false,
  };
  let error: string = "";


  const queryClient = useQueryClient();


  const addMutation = createMutation({
    mutationFn: async (post: Post): Promise<AxiosResponse> => {
      return api.post("posts", post);
    },
    onSuccess: () => {
      console.log("Task added");
      post.title = "";
      queryClient.invalidateQueries({ queryKey: ["posts"] });
    },
    onError: (err: AxiosError) => {
      error =
        `Error: ${err.response?.status}, ${err.response?.data}` ||
        err.code ||
        "Unexpected error";
    },
  });

  const editMutation = createMutation({
    mutationFn: async ({
      task,
      id,
    }: {
      task: Post;
      id: string;
    }): Promise<AxiosResponse> => {
      return api.patch(`posts/${id}`, task);
    },
    onSuccess: () => {
      console.log("Task edited");
      queryClient.invalidateQueries({ queryKey: ["posts"] });
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

 //это тоже состояние $: posts
  $: posts = createQuery<{ data: Post[] }, AxiosError>({
    queryKey: ["posts"],
    queryFn: async () => await api.get("/posts"),
  });
</script>

<main class="max-w-4xl mx-auto flex flex-col items-center justify-center">
  <h1 class="text-3xl mt-2 max-w-36 font-semibold">To Do List</h1>
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
  {#if $posts.error}
    <p class="text-red-500"
      >{`Error: ${$posts.error.response?.status || $posts.error.message}`|| "Unexpected error"}</p
    >
  {/if}

  {#if $posts.isLoading}
    <p>Loading tasks...</p>
  {/if}

  <div class="flex flex-col gap-y-2">
    {#if $posts.isSuccess}
      {#each $posts.data.data as item}
        {#key item.id}
          <form
            on:submit={(e) => {
              e.preventDefault();
              e.stopPropagation();
              const updatedTask = { ...item, title: item.title };
              $editMutation.mutate({ task: updatedTask, id: item.id! });
            }}
          >
            <Input bind:value={item.title} />
            <button
              class="p-2 border-[1px] border-gray-950 rounded-md mt-3 mb-4"
            >
              Edit
            </button>
          </form>
        {/key}
      {/each}
    {/if}
  </div>
</main>
