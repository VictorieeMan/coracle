<script lang="ts">
  import type {ComponentType, SvelteComponentTyped} from "svelte"
  import {Route} from "svelte-routing"
  import {base64DecodeOrPlainWebSocketURL} from "src/util/misc"
  import Notifications from "src/app/views/Notifications.svelte"
  import Bech32Entity from "src/app/views/Bech32Entity.svelte"
  import ChatDetail from "src/app/views/ChatDetail.svelte"
  import ChatList from "src/app/views/ChatList.svelte"
  import Feeds from "src/app/views/Feeds.svelte"
  import UserKeys from "src/app/views/UserKeys.svelte"
  import Apps from "src/app/views/Apps.svelte"
  import Logout from "src/app/views/Logout.svelte"
  import MessagesDetail from "src/app/views/MessagesDetail.svelte"
  import MessagesList from "src/app/views/MessagesList.svelte"
  import NotFound from "src/app/views/NotFound.svelte"
  import PersonDetail from "src/app/views/PersonDetail.svelte"
  import RelayDetail from "src/app/views/RelayDetail.svelte"
  import RelayList from "src/app/views/RelayList.svelte"
  import UserProfile from "src/app/views/UserProfile.svelte"
  import UserSettings from "src/app/views/UserSettings.svelte"
  import {storage} from "src/app/engine"

  const TypedRoute = Route as ComponentType<SvelteComponentTyped>

  let ready = false

  storage.ready.then(() => {
    ready = true
  })
</script>

{#if ready}
  <div class="pt-16 text-gray-2 lg:ml-48">
    <TypedRoute path="/notifications" component={Notifications} />
    <TypedRoute path="/notifications/:activeTab" component={Notifications} />
    <TypedRoute path="/notes" let:params>
      <Feeds />
    </TypedRoute>
    <TypedRoute path="/people/:npub" let:params>
      {#key params.npub}
        <PersonDetail npub={params.npub} />
      {/key}
    </TypedRoute>
    <TypedRoute path="/chat" component={ChatList} />
    <TypedRoute path="/chat/:entity" let:params>
      {#key params.entity}
        <ChatDetail entity={params.entity} />
      {/key}
    </TypedRoute>
    <TypedRoute path="/conversations">
      <MessagesList activeTab="conversations" />
    </TypedRoute>
    <TypedRoute path="/requests">
      <MessagesList activeTab="requests" />
    </TypedRoute>
    <TypedRoute path="/conversations/:entity" let:params>
      {#key params.entity}
        <MessagesDetail entity={params.entity} />
      {/key}
    </TypedRoute>
    <TypedRoute path="/apps" component={Apps} />
    <TypedRoute path="/keys" component={UserKeys} />
    <TypedRoute path="/relays" component={RelayList} />
    <TypedRoute path="/relays/:b64OrUrl" let:params>
      {#key params.b64url}
        <RelayDetail url={base64DecodeOrPlainWebSocketURL(params.b64OrUrl)} />
      {/key}
    </TypedRoute>
    <TypedRoute path="/profile" component={UserProfile} />
    <TypedRoute path="/settings" component={UserSettings} />
    <TypedRoute path="/logout" component={Logout} />
    <TypedRoute path="/:entity" let:params>
      {#key params.entity}
        <Bech32Entity entity={params.entity} />
      {/key}
    </TypedRoute>
    <TypedRoute path="*" component={NotFound} />
  </div>
{/if}
