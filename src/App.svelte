<main>
    {#if supported}
      <TableSearch shadow striped placeholder="Search by family" hoverable={true} bind:inputValue={searchTerm}>
        <caption class="p-5 text-lg font-semibold text-left text-gray-900 bg-white dark:text-white dark:bg-gray-800">
          <p class="mt-1 text-sm font-normal text-gray-500 dark:text-gray-400">如果首次使用，请 <A on:click={
            async () => {
              await window.queryLocalFonts({force: true});
              fonts = await window.queryLocalFonts();
            }
          }>点击此处</A> 授权获取本地字体列表。</p>
          <p class="mt-1 text-sm font-normal text-gray-500 dark:text-gray-400">如果在其他工具中字体名不生效，请复制 CSS 添加到自定义样式中。</p>
        </caption>
        <TableHead defaultRow={false}>
          <TableHeadCell>FAMILY</TableHeadCell>
          <TableHeadCell>FULL NAME</TableHeadCell>
          <TableHeadCell>STYLE</TableHeadCell>
          <TableHeadCell>POSTSCRIPT NAME</TableHeadCell>
          <TableHeadCell>PREVIEW</TableHeadCell>
          <TableHeadCell>OPERATION</TableHeadCell>
        </TableHead>
        <TableBody tableBodyClass="divide-y">
          {#each filtered_fonts as font}
            <TableBodyRow>
              <TableBodyCell>{font.family}</TableBodyCell>
              <TableBodyCell>{font.fullName}</TableBodyCell>
              <TableBodyCell>{font.style}</TableBodyCell>
              <TableBodyCell>{font.postscriptName}</TableBodyCell>
              <TableBodyCell>
                <span style="font-family: {font.postscriptName};">{preview_text}</span>
              </TableBodyCell>
              <TableBodyCell>
                <ButtonGroup class="*:!ring-primary-700">
                  <Button on:click={() => navigator.clipboard.writeText(font.postscriptName)}>Copy Name</Button>
                  <Button on:click={() => navigator.clipboard.writeText(handleCSS(font))}>Copy CSS</Button>
                </ButtonGroup>
              </TableBodyCell>
            </TableBodyRow>
          {/each}
        </TableBody>
      </TableSearch>
    {:else}
      <p>不支持本地字体查询，请使用 Chrome/Edge 浏览器。</p>
    {/if}
</main>


<script>
import "./app.css";
import { Button, ButtonGroup, Table, TableSearch, TableBody, TableBodyCell, TableBodyRow, TableHead, TableHeadCell, A } from 'flowbite-svelte';

// Check queryLocalFonts API window.queryLocalFonts()
let supported = typeof window.queryLocalFonts === "function";
let css = "";

let searchTerm = "";
let fonts = [];
$: filtered_fonts = fonts.filter((item) => item.family.toLowerCase().indexOf(searchTerm.toLowerCase()) !== -1);

if (supported) {
    window.queryLocalFonts().then(local_fonts => {
      fonts = local_fonts;
      // load fonts with @font-face
      css = fonts.map(font => {
        return `
          @font-face {
            font-family: '${font.postscriptName}';
            src: local('${font.postscriptName}'), local('${font.fullName}');
          }
        `;
      }).join("");
      const style = document.createElement("style");
      style.innerHTML = css;
      document.head.appendChild(style);
    });
}

let preview_text = "橘子是最完美的水果～";
let preview_font = "Arial";

function handleCSS(font) {
  return `
    @font-face {
      font-family: '${font.postscriptName}';
      src: local('${font.postscriptName}'), local('${font.fullName}');
    }
  `;
}

</script>

<style>
main {
  padding: 0 4rem;
}
</style>
