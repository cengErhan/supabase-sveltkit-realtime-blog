<script>
  import supabase from '$lib/supabase';
  import { readable, get } from 'svelte/store';

  async function addPost(e) {
    const formData = new FormData(e.target);
    const title = formData.get('title');
    const content = formData.get('content');
    const { data, error } = await supabase
      .from('posts')
      .insert({ title: title, content: content });
  }

  const posts = readable(null, (set) => {
    supabase
      .from('posts')
      .select('*')
      .then(({ error, data }) => set(data));

    //add subscription to supabase
    const subscription = supabase
      .from('posts')
      .on('*', (payload) => {
        if (payload.eventType == 'INSERT') {
          set([...get(posts), payload.new]);
        }
      })
      .subscribe();

    return () => supabase.removeSubscription(subscription);
  });
  
</script>

<main>
  <h1>Welcome to our blog!</h1>
  <h2>Make a post</h2>
  <div class="form">
    <form on:submit|preventDefault={addPost}>
      <input type="text" name="title" id="title" placeholder="Title" /><br /><br
      />
      <textarea
        type="text"
        name="content"
        id="content"
        placeholder="Content"
      /><br /><br />
      <button>Send Post</button>
    </form>
  </div>
  <h2>See your Posts</h2>
  <div class="posts">
    {#if $posts}
      {#each $posts as { title, content }}
        <div>
          <h3>{title}</h3>
          <p>{content}</p>
        </div>
      {:else}
        <h3>You don't have any posts</h3>
      {/each}
    {:else}
      <p>Loading ...</p>
    {/if}
  </div>
</main>

<style>
  main {
    display: flex;
    flex-direction: column;
    text-align: center;
    align-items: center;
    justify-content: center;
  }
  .posts {
    display: flex;
    flex-wrap: wrap;
    border: 1px solid rebeccapurple;
    width: 80%;
    text-align: center;
    justify-content: center;
    align-items: center;
  }
  .posts p {
    color: olive;
  }
  .posts h3 {
    color: darkorange;
  }
  .posts div {
    border: 1px solid red;
    padding: 1.5rem;
  }
</style>
