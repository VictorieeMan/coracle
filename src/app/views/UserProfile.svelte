<script lang="ts">
  import {fly} from "src/util/transition"
  import {navigate} from "svelte-routing"
  import Input from "src/partials/Input.svelte"
  import ImageInput from "src/partials/ImageInput.svelte"
  import Textarea from "src/partials/Textarea.svelte"
  import Anchor from "src/partials/Anchor.svelte"
  import Content from "src/partials/Content.svelte"
  import Heading from "src/partials/Heading.svelte"
  import {Directory, Keys, user, Builder} from "src/app/engine"
  import {routes} from "src/app/state"
  import {publishWithToast} from "src/app/state"

  const nip05Url = "https://github.com/nostr-protocol/nips/blob/master/05.md"
  const lud16Url = "https://blog.getalby.com/create-your-lightning-address/"
  const pseudUrl =
    "https://www.coindesk.com/markets/2020/06/29/many-bitcoin-developers-are-choosing-to-use-pseudonyms-for-good-reason/"

  const submit = async event => {
    const relays = user.getRelayUrls("write")

    event?.preventDefault()
    publishWithToast(Builder.setProfile(values), relays)
    navigate(routes.person($pubkey))
  }

  const {pubkey} = Keys

  let values = Directory.getProfile($pubkey)

  document.title = "Profile"
</script>

<form on:submit={submit} in:fly={{y: 20}}>
  <Content>
    <div class="mb-4 flex flex-col items-center justify-center">
      <Heading>About You</Heading>
      <p>
        Give people a friendly way to recognize you. We recommend you do not use your real name or
        share your personal information. The future of the internet is
        <Anchor class="underline" external href={pseudUrl}>pseudonymous</Anchor>.
      </p>
    </div>
    <div class="flex w-full flex-col gap-8">
      <div class="flex flex-col gap-1">
        <strong>username</strong>
        <Input type="text" name="name" wrapperClass="flex-grow" bind:value={values.name}>
          <i slot="before" class="fa-solid fa-user-astronaut" />
        </Input>
        <p class="text-sm text-gray-1">
          Your username can be changed at any time. To prevent spoofing, a few characters of your
          public key will also be displayed next to your posts.
        </p>
      </div>
      <div class="flex flex-col gap-1">
        <strong>NIP-05 Identifier</strong>
        <Input type="text" name="name" wrapperClass="flex-grow" bind:value={values.nip05}>
          <i slot="before" class="fa-solid fa-user-check" />
        </Input>
        <p class="text-sm text-gray-1">
          Enter a <Anchor class="underline" external href={nip05Url}>NIP-05</Anchor> address to verify
          your public key.
        </p>
      </div>
      <div class="flex flex-col gap-1">
        <strong>Lightning address</strong>
        <Input type="text" name="name" wrapperClass="flex-grow" bind:value={values.lud16}>
          <i slot="before" class="fa-solid fa-bolt" />
        </Input>
        <p class="text-sm text-gray-1">
          Enter a <Anchor class="underline" external href={lud16Url}>LUD-16</Anchor> address to enable
          sending and receiving lightning tips (LUD-06 will also work).
        </p>
      </div>
      <div class="flex flex-col gap-1">
        <strong>About you</strong>
        <Textarea name="about" bind:value={values.about} />
        <p class="text-sm text-gray-1">
          Tell the world about yourself. This will be shown on your profile page.
        </p>
      </div>
      <div class="flex flex-col gap-1">
        <strong>Profile Picture</strong>
        <ImageInput
          bind:value={values.picture}
          icon="image-portrait"
          maxWidth={480}
          maxHeight={480} />
        <p class="text-sm text-gray-1">Please be mindful of others and only use small images.</p>
      </div>
      <div class="flex flex-col gap-1">
        <strong>Profile Banner</strong>
        <ImageInput bind:value={values.banner} icon="panorama" />
        <p class="text-sm text-gray-1">
          In most clients, this image will be shown on your profile page.
        </p>
      </div>
      <Anchor tag="button" theme="button" type="submit" class="text-center">Save</Anchor>
    </div>
  </Content>
</form>
