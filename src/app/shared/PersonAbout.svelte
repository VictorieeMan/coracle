<script lang="ts">
  import {ellipsize} from "hurdak"
  import {parseContent} from "src/util/notes"
  import Anchor from "src/partials/Anchor.svelte"
  import {Directory} from "src/app/engine"

  export let pubkey
  export let truncate = false

  const profile = Directory.getProfile(pubkey)
  const about = profile.about || ""
  const content = parseContent({content: truncate ? ellipsize(about, 140) : about})
</script>

<p>
  {#each content as { type, value }}
    {#if type === "newline"}
      {#each value as _}
        <br />
      {/each}
    {:else if type === "link"}
      <Anchor class="underline" external href={value.url}>
        {value.url.replace(/https?:\/\/(www\.)?/, "")}
      </Anchor>
    {:else if type.startsWith("nostr:")}
      <Anchor class="underline" external href={"/" + value.entity}>
        {#if value.pubkey}
          {Directory.displayProfile(profile)}
        {:else if value.id}
          event {value.id}
        {:else}
          {value.entity.slice(0, 10) + "..."}
        {/if}
      </Anchor>
    {:else}
      {value}
    {/if}
  {/each}
</p>
