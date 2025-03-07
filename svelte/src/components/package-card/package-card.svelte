<script lang="ts">
  import "$appcss";
  import ProgressCircle from "$components/progress-circle/progress-circle.svelte";
  import { PackageStates, type GUIPackage } from "$libs/types";
  import { findRecentInstalledVersion, packageWasUpdated } from "$libs/packages/pkg-utils";
  import BgImage from "./bg-image.svelte";
  import PackageInstallButton from "$components/package-install-button/package-install-button.svelte";
  import PackageInstalledBadge from "$components/package-install-button/package-installed-badge.svelte";
  import { getPackageName } from "$libs/packages/pkg-utils";
  import InstallResultOverlay from "$components/install-result-overlay/install-result-overlay.svelte";

  export let pkg: GUIPackage;
  export let link: string;
  export let progessLoading = 0;
  export let tight = false;

  export let layout: "bottom" | "right" | "left" = "bottom";

  export let onClickCTA = async () => {
    console.log("do nothing");
  };

  // Using this instead of css :active because there is a button inside of a button
  let isActive = false;
  const activate = () => (isActive = true);
  const deactivate = () => (isActive = false);

  const preventPropagation = (evt: MouseEvent) => evt.stopPropagation();
</script>

<div class="relative">
  <section
    class="package-card border-gray relative h-auto border {layout} {tight ? 'tight' : ''}"
    class:active={isActive}
    class:updated={packageWasUpdated(pkg)}
  >
    <BgImage
      class="absolute left-0 top-0 h-full w-full"
      {layout}
      project={pkg.full_name}
      url={pkg.image_512_url}
      cachedImageUrl={pkg.cached_image_url}
      hasImage={!!pkg.image_added_at}
    />

    <a href={link} on:mousedown={activate} on:mouseup={deactivate} on:mouseleave={deactivate}>
      <div
        data-testid={`package-card-${pkg.slug}`}
        class="package-card-content absolute h-full w-full flex-col justify-between"
      >
        <div class="hint-container">
          <div class="hint">
            <div class="line-clamp-1 text-xs">view more details</div>
            <div class="hint-icon"><i class="icon-upward-arrow" /></div>
          </div>
        </div>
        <div class="content-container absolute bottom-0 w-full {layout} {tight ? 'tight' : ''}">
          <article class="card-thumb-label">
            {#if layout === "bottom"}
              <div class="flex items-center">
                <h3
                  class="text-bold font-mona line-clamp-1 break-all text-2xl font-bold text-white {tight
                    ? 'text-1xl'
                    : ''}"
                >
                  {getPackageName(pkg)}
                </h3>
                {#if pkg.state === PackageStates.INSTALLED}
                  <i class="icon-check-circle-o mb-1 ml-2 flex text-2xl text-[#00ffd0]" />
                {/if}
              </div>
              <p class="line-clamp-2 h-[32px] text-xs font-thin lowercase">
                {pkg.short_description ?? ""}
              </p>
            {:else}
              <div class="mb-4 flex items-center">
                <h3
                  class="text-bold font-mona line-clamp-1 text-3xl font-bold {tight
                    ? 'line-clamp-3 pt-8 text-2xl'
                    : ''}"
                >
                  {getPackageName(pkg)}
                </h3>
                {#if pkg.state === PackageStates.INSTALLED}
                  <i class="icon-check-circle-o mb-1 ml-2 flex text-3xl text-[#00ffd0]" />
                {/if}
              </div>
              <p
                class="text-[14px] font-thin lowercase leading-[20px] {tight
                  ? 'line-clamp-[5] h-[120px]'
                  : 'line-clamp-[8] h-[160px]'}"
              >
                {pkg.description ?? ""}
              </p>
            {/if}
          </article>
          <div class="mt-3.5 w-full">
            <div class="flex w-fit flex-col items-center">
              <div class="install-button {layout}" on:mousedown={preventPropagation}>
                {#if pkg.state === PackageStates.INSTALLED}
                  <PackageInstalledBadge {pkg} />
                {:else}
                  <PackageInstallButton
                    {pkg}
                    onClick={(evt) => {
                      // prevent default to prevent the link that this button is inside of from being followed
                      evt?.preventDefault();
                      onClickCTA();
                    }}
                  />
                {/if}
              </div>
              <div class="mt-1.5 h-[10px] leading-[10px]">
                {#if pkg.state === "NEEDS_UPDATE"}
                  <span class="text-[10px]">
                    <span class="opacity-70">you have</span>
                    v{findRecentInstalledVersion(pkg)}
                  </span>
                {/if}
              </div>
            </div>
          </div>
        </div>
      </div>
    </a>

    {#if progessLoading > 0 && progessLoading < 100}
      <div class="absolute left-0 top-0 z-40 h-full w-full bg-black bg-opacity-50">
        <div class="absolute left-0 right-0 top-1/2 m-auto -mt-12 h-24 w-24">
          <ProgressCircle value={progessLoading} />
        </div>
      </div>
    {/if}
  </section>
  <InstallResultOverlay {pkg} />
</div>

<style>
  section {
    transition: all 0.3s;
    width: 100%;
    height: 340px;
    background-size: cover;
    box-sizing: border-box;
  }

  section.active::before {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(26, 26, 26, 0.7);
    z-index: 2;
    content: "";
    pointer-events: none;
  }

  section.package-card:active {
    border-color: #8000ff;
    box-shadow: 0px 0px 0px 2px rgba(128, 0, 255, 0.5);
  }

  section.package-card.updated {
    border-color: #00ffd0;
  }

  section.package-card.updated:active {
    border-color: #00ffd0;
    box-shadow: 0px 0px 0px 2px rgba(0, 255, 208, 0.5);
  }

  .content-container {
    height: 50%;
    background: linear-gradient(180deg, rgba(26, 26, 26, 0.3) 0%, rgba(26, 26, 26, 0.75) 72.92%);
    display: flex;
    flex-direction: column;
    padding: 28px 14px;
    justify-content: center;
  }

  .content-container.tight {
    color: rgb(14, 14, 14, 0.9);
    background: linear-gradient(
      180deg,
      rgba(224, 224, 224, 0.3) 0%,
      rgba(224, 224, 224, 0.75) 72.92%
    );
  }

  .content-container.bottom {
    left: 0px;
  }

  .content-container.left {
    height: 100%;
    width: 60%;
    left: 0px;
    padding: 28px 28px;
  }

  .content-container.left.tight {
    width: 33%;
  }

  .content-container.right {
    height: 100%;
    width: 60%;
    right: 0px;
    padding: 28px 28px;
  }

  .hint-container {
    position: absolute;
    top: 0px;
    right: 0px;
    display: flex;
    justify-content: flex-end;
    z-index: 1;
  }

  .hint {
    min-width: 240px;
    padding-left: 30%;
    height: 24px;
    display: flex;
    align-items: center;
    justify-content: flex-end;
    column-gap: 0.5rem;
    background: linear-gradient(270deg, #e1e1e1 66.29%, rgba(225, 225, 225, 0) 100%);
    color: #1a1a1a;
    visibility: hidden;
  }

  .hint-icon {
    background: #8000ff;
    height: 24px;
    width: 24px;
    display: flex;
    justify-content: center;
    align-items: center;
    color: #e1e1e1;
  }

  .package-card.right {
    min-width: 550px;
  }

  .package-card.left {
    min-width: 550px;
  }

  .package-card.left.tight {
    min-width: 300px;
    border-radius: 4px;
    overflow: hidden;
  }

  .package-card:hover .hint {
    visibility: visible;
  }

  .card-thumb-label {
    text-align: left;
    width: 100%;
  }

  .install-button {
    min-width: 160px;
    width: fit-content;
  }
</style>
