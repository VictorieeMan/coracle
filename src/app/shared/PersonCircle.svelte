<script lang="ts">
  import cx from "classnames"
  import {defaultTo} from "ramda"
  import {stringToHue, hsl} from "src/util/misc"
  import ImageCircle from "src/partials/ImageCircle.svelte"
  import LogoSvg from "src/partials/LogoSvg.svelte"
  import {Directory} from "src/app/engine"

  export let pubkey
  export let size = 4

  const hue = stringToHue(pubkey)
  const primary = hsl(hue, {lightness: 80})
  const secondary = hsl(hue, {saturation: 30, lightness: 30})
  const profile = Directory.profiles.key(pubkey).derived(defaultTo({pubkey}))
</script>

{#if $profile.picture}
  <ImageCircle {size} src={$profile.picture} class={$$props.class} />
{:else}
  <div
    class={cx(
      $$props.class,
      `relative overflow-hidden w-${size} h-${size} inline-block shrink-0 rounded-full border border-solid
       border-gray-1 bg-cover bg-center`
    )}
    style="--logo-color: {primary}; --logo-bg-color: {secondary}; background-color: var(--logo-bg-color);">
    <LogoSvg
      class="logo absolute left-2/4 top-2/4 -translate-x-1/2 -translate-y-1/2"
      style="height: 85%; width: 85%;" />
  </div>
{/if}
