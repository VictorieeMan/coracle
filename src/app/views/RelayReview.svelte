<script lang="ts">
  import {fly} from "src/util/transition"
  import {modal} from "src/partials/state"
  import Anchor from "src/partials/Anchor.svelte"
  import Content from "src/partials/Content.svelte"
  import Heading from "src/partials/Heading.svelte"
  import Compose from "src/app/shared/Compose.svelte"
  import Rating from "src/partials/Rating.svelte"
  import {Builder, Outbox, user} from "src/app/engine"

  export let url

  let compose
  let rating

  const onSubmit = () => {
    const review = compose.parse()
    const event = Builder.createLabel({
      content: review,
      tagClient: false,
      tags: [
        ["L", "social.coracle.ontology"],
        ["l", "review/relay", "social.coracle.ontology", JSON.stringify({quality: rating})],
        ["r", url],
      ],
    })

    Outbox.publish(event, user.getRelayUrls("write"))

    modal.pop()
  }
</script>

<form on:submit|preventDefault={onSubmit} in:fly={{y: 20}}>
  <Content size="lg">
    <Heading class="text-center">Leave a review</Heading>
    <div class="flex w-full flex-col gap-4">
      <div class="flex flex-col gap-1">
        <div class="flex items-center justify-between gap-2">
          <strong>Your rating:</strong>
          <Rating bind:value={rating} />
        </div>
      </div>
      <Compose
        {onSubmit}
        class="shadow-inset rounded bg-input px-2 py-2 text-black"
        style="min-height: 6rem"
        bind:this={compose} />
      <Anchor tag="button" theme="button" type="submit" class="flex-grow text-center">Send</Anchor>
    </div>
  </Content>
</form>
