<script>
  import {onMount} from "svelte"
  import {groupBy} from "ramda"
  import {mapVals} from "hurdak"
  import {fuzzy} from "src/util/misc"
  import {normalizeRelayUrl, Tags, getAvgQuality} from "src/util/nostr"
  import Input from "src/partials/Input.svelte"
  import RelayCard from "src/app/shared/RelayCard.svelte"
  import {Nip65, Network, Keys, user} from "src/app/engine"

  export let q = ""
  export let limit = 50
  export let placeholder = "Search relays or add a custom url"
  export let hideIfEmpty = false

  let search
  let reviews = []

  const joined = Nip65.policies.key(Keys.pubkey.get()).derived(() => new Set(user.getRelayUrls()))
  const knownRelays = Nip65.relays

  $: ratings = mapVals(
    events => getAvgQuality("review/relay", events),
    groupBy(e => Tags.from(e).getMeta("r"), reviews)
  )

  $: {
    search = fuzzy(
      $knownRelays.filter(r => !$joined.has(r.url)),
      {keys: ["name", "description", "url"]}
    )
  }

  onMount(() => {
    const sub = Network.subscribe({
      relays: Nip65.getPubkeyHints(3, Keys.pubkey.get(), "read"),
      filter: {
        limit: 1000,
        kinds: [1985],
        "#l": ["review/relay"],
        "#L": ["social.coracle.ontology"],
      },
      onEvent: event => {
        reviews = reviews.concat(event)
      },
    })

    return sub.close
  })
</script>

<div class="flex flex-col gap-4">
  <Input bind:value={q} type="text" wrapperClass="flex-grow" {placeholder}>
    <i slot="before" class="fa-solid fa-search" />
  </Input>
  <div class="flex flex-col gap-2">
    {#if q.match("^.+\\..+$")}
      <RelayCard relay={{url: normalizeRelayUrl(q)}} />
    {/if}
    {#each !q && hideIfEmpty ? [] : search(q).slice(0, limit) as relay (relay.url)}
      <slot name="item" {relay}>
        <RelayCard rating={ratings[relay.url]} {relay} />
      </slot>
    {/each}
    <slot name="footer">
      <small class="text-center">
        Showing {Math.min(($knownRelays || []).length - $joined.size, 50)}
        of {($knownRelays || []).length - $joined.size} known relays
      </small>
    </slot>
  </div>
</div>
