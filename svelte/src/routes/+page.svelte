<script lang="ts">
  import "$appcss";

  import { page } from "$app/stores";
  import { t } from "$libs/translations";
  import { afterNavigate } from "$app/navigation";
  import { packagesStore } from "$libs/stores";
  import Packages from "$components/packages/packages.svelte";
  import DiscoverPackages from "$components/discover-packages/discover-carousel.svelte";
  import { PackageStates, SideMenuOptions, type GUIPackage } from "$libs/types";
  // import SortingButtons from "$components/search-packages/sorting-buttons.svelte";
  import SideMenu from "$components/side-menu/side-menu.svelte";
  import NotificationBar from "$components/notification-bar/notification-bar.svelte";
  import Button from "$components/button/button.svelte";
  import log from "$libs/logger";
  import { formatPercent } from "$components/lib/percent";

  const { packageList } = packagesStore;

  const url = $page.url;

  $: sideMenuOption = (url.searchParams.get("tab") as SideMenuOptions) || SideMenuOptions.discover;

  let updating = false;

  let packagesScrollY = 0;
  $: packageCount = 0;
  $: currentUpdatingPkg = $packageList.find((p) => p.state === PackageStates.UPDATING);
  $: updatingMessage = `updating ${currentUpdatingPkg?.full_name} (${formatPercent(
    currentUpdatingPkg?.install_progress_percentage
  )}%)`;

  $: pkgsToUpdate = $packageList.filter((p: GUIPackage) => p.state === PackageStates.NEEDS_UPDATE);
  async function updateAll() {
    updating = true;
    log.info(`updating: ${pkgsToUpdate.length} packages`);
    for (const pkg of pkgsToUpdate) {
      try {
        await packagesStore.installPkg(pkg);
      } catch (error) {
        log.error(error);
      }
    }
    updating = false;
  }

  $: needsUpdateCount = pkgsToUpdate.length;

  afterNavigate(({ to }) => {
    if (to?.url?.pathname === "/") {
      const tab = to.url.searchParams.get("tab");
      sideMenuOption = !tab ? SideMenuOptions.discover : (tab as SideMenuOptions);
    }
  });
</script>

<div class="content-container">
  <div id="content" class="flex flex-col">
    <NotificationBar />
    <article class="relative h-auto w-full flex-grow overflow-hidden">
      <ul class="px-2">
        {#if sideMenuOption == SideMenuOptions.discover}
          <DiscoverPackages bind:scrollY={packagesScrollY} />
        {:else}
          <Packages
            packageFilter={sideMenuOption}
            bind:scrollY={packagesScrollY}
            bind:packageCount
          />
        {/if}
      </ul>
      <header
        class="z-30 flex items-center justify-between"
        class:scrolling={packagesScrollY > 150}
      >
        <h1 class="font-mona pl-9 text-2xl font-bold text-white">
          {$t(`side-menu-title.${sideMenuOption}`).toLowerCase()}
        </h1>
        <!-- 
        <section class="border-gray mt-4 mr-4 h-10 w-48 border rounded-sm">
          
          we might bring it back?
          <SortingButtons onSort={(prop, dir) => {
            sortBy = prop;
            sortDirection = dir;
          }} />
        </section>
        -->
        {#if needsUpdateCount && sideMenuOption === SideMenuOptions.installed_updates_available}
          <!-- 22px right margin to account for the scrollbar on the package cards -->
          <div class="mr-[22px] flex items-center justify-end text-sm">
            {#if currentUpdatingPkg}
              <p class="text-gray px-2 font-mono">{updatingMessage}</p>
            {/if}
            <div>
              <Button
                class="h-8 min-w-[125px] p-2 text-xs"
                loading={updating}
                type="plain"
                color="secondary"
                onClick={updateAll}
              >
                {$t(`package.update-all`)} [{needsUpdateCount}]
              </Button>
            </div>
          </div>
        {/if}
      </header>
    </article>
  </div>

  <SideMenu bind:activeOption={sideMenuOption} />
</div>

<style>
  .content-container {
    height: auto;
    overflow-y: hidden;
    padding-left: 4px;
    padding-right: 4px;
    overflow-x: hidden;
    position: relative;
  }

  #content {
    width: calc(100vw - 211px);
    margin-left: 205px;
    height: calc(100vh - 50px);
    overflow: hidden;
  }

  header {
    position: absolute;
    top: 0px;
    left: 1px;
    height: 72px;
    width: 100%;
    background-image: linear-gradient(rgba(26, 26, 26, 1), rgba(26, 26, 26, 0));
    padding-top: 15px;
  }

  header h1 {
    padding-top: 8px;
  }

  header.scrolling {
    height: 60px;
    background-color: #222222;
    padding-top: 5px;
  }

  header.scrolling h1 {
    padding-top: 0px;
  }

  /* width */
  ::-webkit-scrollbar {
    width: 6px;
  }

  /* Track */
  ::-webkit-scrollbar-track {
    background: #272626;
  }

  /* Handle */
  ::-webkit-scrollbar-thumb {
    background: #949494;
    border-radius: 4px;
  }

  /* Handle on hover */
  ::-webkit-scrollbar-thumb:hover {
    background: white;
  }
</style>
