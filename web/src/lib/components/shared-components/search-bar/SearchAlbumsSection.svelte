<script lang="ts">
  import Combobox, { type ComboBoxOption } from '$lib/components/shared-components/Combobox.svelte';
  import { getAllAlbums, type AlbumResponseDto } from '@immich/sdk';
  import { Text, Icon } from '@immich/ui';
  import { mdiClose } from '@mdi/js';
  import { onMount } from 'svelte';
  import { t } from 'svelte-i18n';
  import type { SvelteSet } from 'svelte/reactivity';

  interface Props {
    selectedAlbums: SvelteSet<string>;
  }

  let { selectedAlbums = $bindable() }: Props = $props();

  let allAlbums: AlbumResponseDto[] = $state([]);
  let albumMap = $derived(Object.fromEntries(allAlbums.map((album) => [album.id, album])));
  let selectedOption = $state<ComboBoxOption | undefined>(undefined);

  onMount(async () => {
    allAlbums = await getAllAlbums({});
  });

  const handleSelect = (option?: ComboBoxOption) => {
    if (!option) {
      return;
    }

    selectedAlbums.add(option.value);
    selectedOption = undefined;
  };

  const handleRemove = (albumId: string) => {
    selectedAlbums.delete(albumId);
  };
</script>

<div id="album-selection">
  <div class="mb-4 flex flex-col">
    <Text class="py-3" fontWeight="medium">{$t('albums')}</Text>
    <Combobox
      hideLabel
      onSelect={handleSelect}
      label={$t('albums')}
      defaultFirstOption
      options={allAlbums.map((album) => ({ id: album.id, label: album.albumName, value: album.id }))}
      bind:selectedOption
      placeholder={$t('search_albums')}
    />
  </div>

  <section class="flex flex-wrap gap-1 pt-2">
    {#each selectedAlbums as albumId (albumId)}
      {@const album = albumMap[albumId]}
      {#if album}
        <div class="group flex transition-all">
          <span
            class="inline-block h-min rounded-s-full bg-primary py-1 ps-3 pe-1 text-center align-baseline leading-none whitespace-nowrap text-gray-100 transition-all group-hover:ps-3 hover:bg-immich-primary/80 dark:text-immich-dark-gray dark:hover:bg-immich-dark-primary/80"
          >
            <p class="text-sm">{album.albumName}</p>
          </span>

          <button
            type="button"
            class="place-content-center place-items-center rounded-e-full bg-immich-primary/95 py-1 ps-1 pe-2 text-gray-100 transition-all hover:bg-immich-primary/80 dark:bg-immich-dark-primary/95 dark:text-immich-dark-gray dark:hover:bg-immich-dark-primary/80"
            title={$t('remove_filter')}
            aria-label={$t('remove_filter')}
            onclick={() => handleRemove(albumId)}
          >
            <Icon icon={mdiClose} />
          </button>
        </div>
      {/if}
    {/each}
  </section>
</div>
