<script lang="ts">
  import type { Package } from "$libs/types";
  import { t } from "$libs/translations";
  import dayjs from "dayjs";
  import relativeTime from "dayjs/plugin/relativeTime";
  import { shellOpenExternal } from "@native";
  dayjs.extend(relativeTime);

  export let pkg: Package;

  const computeFileSize = (bytes: number): string => {
    let n: number = bytes;
    let unit = "bytes";
    let divisor = 1;

    if (n > 1024 ** 3) {
      unit = "GB";
      divisor = 1024 ** 3;
    } else if (n > 1024 ** 2) {
      unit = "MB";
      divisor = 1024 ** 2;
    } else if (n > 1024) {
      unit = "kB";
      divisor = 1024;
    }

    const N = n / divisor;
    // going for 3 significant figures
    const decimals = N < 10 ? 2 : N < 100 ? 1 : 0;

    return `${N.toFixed(decimals)} ${unit}`;
  };
  console.log(pkg);
</script>

<section class="bg-black pt-2">
  <h1 class="text-primary">{$t("common.metadata").toLowerCase()}</h1>
  <ul class="mb-10 flex flex-col gap-2">
    {#if pkg?.bottles?.length}
      <li>
        <span>updated {dayjs().to(dayjs(pkg?.bottles[0].updated_at))}</span>
      </li>
    {/if}
    {#if pkg?.license}
      <li>
        <span>{pkg.license}</span>
      </li>
    {/if}
    {#if pkg?.bottles?.length}
      <li>
        <span>{computeFileSize(pkg?.bottles[0].bytes)}</span>
      </li>
    {/if}
  </ul>
  {#if pkg.homepage}
    <h1 class="text-primary">{$t("common.homepage").toLowerCase()}</h1>
    <ul class="mb-10 flex flex-col gap-2">
      <!-- svelte-ignore a11y-click-events-have-key-events -->
      <li on:click={() => shellOpenExternal(pkg.homepage)}>
        <span class="hover:text-primary cursor-pointer break-all">{pkg.homepage}</span>
      </li>
    </ul>
  {/if}
  {#if pkg.documentation_url}
    <h1 class="text-primary">{$t("common.documentation").toLowerCase()}</h1>
    <ul class="mb-10 flex flex-col gap-2">
      <!-- svelte-ignore a11y-click-events-have-key-events -->
      <li on:click={() => shellOpenExternal(pkg.documentation_url)}>
        <span class="hover:text-primary cursor-pointer">{pkg.documentation_url}</span>
      </li>
    </ul>
  {/if}
  {#if pkg.github_url}
    <h1 class="text-primary">{$t("common.github-repository").toLowerCase()}</h1>
    <ul class="mb-10 flex flex-col gap-2">
      <!-- svelte-ignore a11y-click-events-have-key-events -->
      <li on:click={() => shellOpenExternal(`https://github.com/${pkg.github_url}`)}>
        <span class="hover:text-primary cursor-pointer">{pkg.github_url}</span>
      </li>
    </ul>
  {/if}
  {#if pkg.contributors?.length}
    <h1 class="text-primary">{$t("common.contributors").toLowerCase()}</h1>
    <ul class="mb-10 flex flex-col gap-2">
      {#each (pkg.contributors || []).filter((c) => !c.login.includes("[bot]")) as contributor}
        <!-- svelte-ignore a11y-click-events-have-key-events -->
        <li on:click={() => shellOpenExternal(`https://github.com/${contributor.login}`)}>
          <!-- <figure class="h-5 w-5 bg-gray">
						<img src={contributor.avatar_url} alt={contributor.login} />
					</figure> -->
          <span class="hover:text-primary cursor-pointer">{contributor.login}</span>
        </li>
      {/each}
    </ul>
  {/if}

  <!-- {#if pkg.categories?.length}
		<h1 class="border border-t-0 border-gray p-4 text-primary">CATEGORIES</h1>
		<ul class="border border-t-0 border-gray p-4">
			{#each pkg.categories as category}
				<li class="border border-gray p-4">
					<i class="icon-calendar" />
					<span class="ml-4">{category}</span>
				</li>
			{/each}
		</ul>
	{/if} -->
</section>

<style>
  ul {
    margin-top: 10px;
  }
  ul > li {
    border-top: gray 1px solid;
    min-height: 36px;
    padding-top: 10px;
  }
</style>
