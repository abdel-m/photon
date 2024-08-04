<script lang="ts">
  import Pageination from '$lib/components/ui/Pageination.svelte'
  import { searchParam } from '$lib/util.js'
  import { page } from '$app/stores'
  import ObjectAutocomplete from '$lib/components/lemmy/ObjectAutocomplete.svelte'
  import { profile } from '$lib/auth.js'
  import { userSettings } from '$lib/settings.js'
  import ModlogItemCard from './item/ModlogItemCard.svelte'
  import ModlogItemTable from './item/ModlogItemTable.svelte'
  import { Button, Select } from 'mono-svelte'
  import {
    Bars3BottomRight,
    Icon,
    MagnifyingGlass,
    ViewColumns,
    XMark,
  } from 'svelte-hero-icons'
  import UserAutocomplete from '$lib/components/lemmy/user/UserAutocomplete.svelte'
  import Placeholder from '$lib/components/ui/Placeholder.svelte'
  import { isAdmin } from '$lib/components/lemmy/moderation/moderation.js'
  import { browser } from '$app/environment'

  export let data

  let view = `${
    $userSettings.modlogCardView ?? browser
      ? !window.matchMedia('(min-width: 1600px)').matches
      : false
  }`

  $: $userSettings.modlogCardView =
    view == 'true' ? true : view == 'false' ? false : undefined
</script>

<svelte:head>
  <title>Journal de modération</title>
</svelte:head>

<div class="flex flex-col gap-4">
  <h1 class="font-bold text-2xl">Journal de modération</h1>
  <div class="flex flex-row flex-wrap gap-2">
    <Select
      bind:value={data.type}
      on:change={(e) => searchParam($page.url, 'type', data.type, 'page')}
      class="w-48"
    >
      <span slot="label" class="flex gap-1 items-center">
      <Icon src={Bars3BottomRight} size="15" mini />
      Type
      </span>
      <option value="All">Tous</option>
      <option value="ModRemovePost">Supprimer le poste</option>
      <option value="ModLockPost">Verrouiller le poste</option>
      <option value="ModFeaturePost">Mettre en avant le poste</option>
      <option value="ModRemoveComment">Supprimer le commentaire</option>
      <option value="ModRemoveCommunity">Supprimer la communauté</option>
      <option value="ModBanFromCommunity">Bannir de la communauté</option>
      <option value="ModAddCommunity">Ajouter un modérateur</option>
      <option value="ModTransferCommunity">Transférer la communauté</option>
      <option value="ModAdd">Ajouter un administrateur</option>
      <option value="ModBan">Bannir un administrateur</option>
      <option value="ModHideCommunity">Masquer la communauté</option>
      <option value="AdminPurgePerson">Purger l'utilisateur</option>
      <option value="AdminPurgeCommunity">Purger la communauté</option>
      <option value="AdminPurgePost">Purger le poste</option>
      <option value="AdminPurgeComment">Purger le commentaire</option>
    </Select>

    <Select bind:value={view} class="w-36">
      <span slot="label" class="flex gap-1 items-center">
        <Icon src={ViewColumns} size="15" mini />
        Affichage
      </span>
      <option value="false">Tableau</option>
      <option value="true">Carte</option>
      <option value="undefined">Défaut</option>
    </Select>
  </div>
  <div class="flex flex-col md:flex-row md:items-center gap-2 w-full">
    <ObjectAutocomplete
      placeholder="Filtrer par communauté"
      jwt={$profile?.jwt}
      listing_type="All"
      showWhenEmpty={true}
      label="Communauté"
      class="flex-1"
      q={$page.url.searchParams.get('community') ? 'Selected' : ''}
      on:select={(e) =>
        searchParam(
          $page.url,
          'community',
          e.detail?.id.toString() ?? '',
          'page'
        )}
    />
    <UserAutocomplete
      instance={$page.url.searchParams.get('instance') || undefined}
      placeholder="Filtrer par utilisateur"
      jwt={$profile?.jwt}
      listing_type="All"
      showWhenEmpty={true}
      class="flex-1"
      label="Utilisateur"
      q={$page.url.searchParams.get('user')
        ? data.filters.user ?? 'Selected'
        : ''}
      on:select={(e) =>
        searchParam($page.url, 'user', e.detail?.id.toString() ?? '', 'page')}
    />
    {#if $profile?.user && isAdmin($profile?.user)}
      <UserAutocomplete
        placeholder="Filtrer par modérateur"
        jwt={$profile?.jwt}
        listing_type="All"
        showWhenEmpty={true}
        class="flex-1"
        label="Modérateur"
        q={$page.url.searchParams.get('mod_id')
          ? data.filters.moderator ?? 'Selected'
          : ''}
        on:select={(e) =>
          searchParam(
            $page.url,
            'mod_id',
            e.detail?.id.toString() ?? '',
            'page'
          )}
      />
    {/if}
    <Button
      on:click={() => {
        searchParam(
          $page.url,
          '',
          '',
          'user',
          'moderator',
          'community',
          'instance'
        )
      }}
      size="square-lg"
      class="self-end flex-shrink-0"
      title="Effacer les filtres"
    >
      <Icon src={XMark} size="16" mini />
    </Button>
  </div>
  {#if data.modlog && data.modlog.length > 0}
    {#if $userSettings.modlogCardView ?? !window.matchMedia('(min-width: 1600px)').matches}
      <div class="flex flex-col gap-4">
        {#each data.modlog as modlog}
          <ModlogItemCard item={modlog} />
        {/each}
      </div>
    {:else}
      <div style="width:100%; overflow-x: auto;">
        <table
          class="table overflow-x-auto table-fixed relative"
          style="min-width: 800px;"
        >
          <colgroup class="table-fixed">
            <col width="8%" />
            <col width="18%" />
            <col width="12%" class="overflow-x-auto" />
            <col width="8%" />
            <col width="10%" />
            <col width="18%" />
            <col width="20%" />
            <col width="20%" />
          </colgroup>
          <thead class="text-left sticky top-0">
            <tr class="rounded-t-lg overflow-hidden">
              <th>Temps</th>
              <th>Modérateur</th>
              <th>Action</th>
              <th>Utilisateur</th>
              <th>Communauté</th>
              <th>Contenu</th>
              <th>Raison</th>
              <th>Lien</th>
           </tr>
          </thead>
          <tbody class="text-sm">
            {#each data.modlog as modlog}
              <ModlogItemTable item={modlog} />
            {/each}
          </tbody>
        </table>
      </div>
    {/if}
    {#if data.modlog.length >= 40}
      <Pageination
        page={data.page}
        on:change={(e) => searchParam($page.url, 'page', e.detail.toString())}
      />
    {/if}
  {:else}
    <Placeholder
      title="Aucun résultat"
      description="Il n'y a aucun journal de modération correspondant à ce filtre. Essayez d'affiner votre recherche."
      icon={MagnifyingGlass}
    />
  {/if}
</div>

<style lang="postcss">
  :global(.table thead tr th) {
    @apply border border-slate-200 dark:border-zinc-800 px-4 py-2 bg-slate-100;
  }

  :global(.table tr td) {
    @apply border border-slate-200 px-4 py-2 overflow-auto;
  }

  :global(.dark .table tr td) {
    @apply border-zinc-800;
  }

  :global(.dark thead tr th) {
    @apply border-zinc-800 bg-zinc-900;
  }
</style>
