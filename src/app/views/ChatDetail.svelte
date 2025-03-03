<script lang="ts">
  import {onMount, onDestroy} from "svelte"
  import {defaultTo, filter, whereEq} from "ramda"
  import {formatTimestamp} from "src/util/misc"
  import {toHex} from "src/util/nostr"
  import {modal} from "src/partials/state"
  import Channel from "src/partials/Channel.svelte"
  import Anchor from "src/partials/Anchor.svelte"
  import ImageCircle from "src/partials/ImageCircle.svelte"
  import PersonBadgeSmall from "src/app/shared/PersonBadgeSmall.svelte"
  import NoteContent from "src/app/shared/NoteContent.svelte"
  import {Builder, Settings, Nip28, user, Keys, Outbox, Nip65, Network} from "src/app/engine"

  export let entity

  const id = toHex(entity)
  const channel = Nip28.channels.key(id).derived(defaultTo({id}))
  const messages = Nip28.messages.derived(filter(whereEq({channel: id})))
  const getRelays = () =>
    Nip65.selectHints(Settings.getSetting("relay_limit"), $channel.hints || [])

  user.setChannelLastChecked(id)

  const join = () => user.joinChannel($channel.id)

  const leave = () => user.leaveChannel($channel.id)

  const edit = () => {
    modal.push({type: "channel/edit", channel: $channel})
  }

  const sendMessage = async content => {
    const relays = getRelays()

    await Outbox.publish(Builder.createChatMessage(id, content, relays[0]), relays)
  }

  onMount(() => {
    const sub = Network.subscribe({
      relays: getRelays(),
      filter: [
        {kinds: [40], ids: [id]},
        {kinds: [41, 42], "#e": [id]},
      ],
    })

    return () => sub.close()
  })

  onDestroy(() => {
    if (!$channel.joined) {
      Nip28.messages.reject(m => m.channel === id)
    }
  })

  $: picture = Settings.imgproxy($channel.picture, {w: 96, h: 96})

  document.title = $channel.name || "Coracle Chat"
</script>

<Channel {messages} {sendMessage}>
  <div slot="header" class="flex h-16 items-start gap-4 overflow-hidden p-2">
    <div class="flex items-center gap-4 pt-1">
      <Anchor type="unstyled" class="fa fa-arrow-left cursor-pointer text-2xl" href="/chat" />
      <ImageCircle size={10} src={picture} />
    </div>
    <div class="flex h-12 flex-grow flex-col pt-px">
      <div class="font-bold">{$channel.name || ""}</div>
      <div>{$channel.about || ""}</div>
    </div>
    <div class="flex h-12 flex-col pt-px">
      <div class="flex w-full items-center justify-between">
        <div class="flex gap-2">
          {#if $channel.pubkey === Keys.pubkey.get()}
            <button class="cursor-pointer text-sm" on:click={edit}>
              <i class="fa-solid fa-edit" /> Edit
            </button>
          {/if}
          {#if $channel.joined}
            <Anchor theme="button" killEvent class="flex items-center gap-2" on:click={leave}>
              <i class="fa fa-right-from-bracket" />
              <span>Leave</span>
            </Anchor>
          {:else if Keys.canSign.get()}
            <Anchor theme="button" killEvent class="flex items-center gap-2" on:click={join}>
              <i class="fa fa-right-to-bracket" />
              <span>Join</span>
            </Anchor>
          {/if}
        </div>
      </div>
    </div>
  </div>
  <div slot="message" let:message>
    {#if message.showProfile}
      <div class="flex items-center justify-between gap-4">
        <PersonBadgeSmall pubkey={message.pubkey} />
        <p class="text-sm text-gray-1">{formatTimestamp(message.created_at)}</p>
      </div>
    {/if}
    <div class="my-1 ml-6 overflow-hidden text-ellipsis">
      <NoteContent showEntire note={message} />
    </div>
  </div>
</Channel>
