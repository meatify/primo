<script>
  import { cloneDeep, isEqual } from 'lodash-es';
  import { Tabs } from '../../components/misc';
  import { CodeMirror } from '../../components';
  import { convertFieldsToData } from '../../utils';
  import ModalHeader from './ModalHeader.svelte';
  import { processors } from '../../component';

  import modal from '../../stores/app/modal';
  import { getAllFields } from '../../stores/helpers';
  import { html as pageHTML, id } from '../../stores/app/activePage';
  import { saved } from '../../stores/app/misc';
  import { html as siteHTML } from '../../stores/data/draft';
  import {
    updateSiteHTML,
    updateActivePageHTML,
  } from '../../stores/actions';

  let localPageHTML = cloneDeep($pageHTML);
  let localSiteHTML = cloneDeep($siteHTML);

  const tabs = [
    {
      id: 'page',
      label: 'Page',
      icon: 'square',
    },
    {
      id: 'site',
      label: 'Site',
      icon: 'th',
    },
  ];

  let activeTab = tabs[0];

  async function saveFinalHTML() {
    updateActivePageHTML(localPageHTML);
    updateSiteHTML(localSiteHTML);
    $saved = false;
  }
</script>

<ModalHeader
  icon="fab fa-html5"
  title="HTML"
  button={{
    label: `Draft`,
    icon: 'fas fa-check',
    onclick: () => {
      saveFinalHTML();
      modal.hide();
    },
  }}
  warn={() => {
    if (
      !isEqual(localPageHTML, $pageHTML) ||
      !isEqual(localSiteHTML, $siteHTML)
    ) {
      const proceed = window.confirm(
        'Undrafted changes will be lost. Continue?'
      );
      return proceed;
    } else return true;
  }}
/>

<main>
  <Tabs {tabs} bind:activeTab />
  <div class="editors">
    {#if activeTab.id === 'page'}
      <span class="head">{'<head>'}</span>
      <CodeMirror
        bind:value={localPageHTML.head}
        style="height:10rem"
        mode="html"
      />

      <span class="before-body">{'Before </body>'}</span>
      <CodeMirror
        bind:value={localPageHTML.below}
        style="height:15rem"
        mode="html"
      />
    {:else}
      <span class="head">{'<head>'}</span>
      <CodeMirror
        bind:value={localSiteHTML.head}
        style="height:10rem"
        mode="html"
      />

      <span class="before-body">{'Before </body>'}</span>
      <CodeMirror
        bind:value={localSiteHTML.below}
        style="height:15rem"
        mode="html"
      />
    {/if}
  </div>
</main>

<style lang="postcss">
  main {
    background: var(--primo-color-black);
    display: flex;
    flex-direction: column;
    padding: 0.5rem;

    .editors {
      flex: 1;

      .head {
        margin-bottom: 0.25rem;
        display: inline-block;
        font-weight: 600;
        color: var(--color-gray-2);
      }

      .before-body {
        margin-bottom: 0.25rem;
        margin-top: 0.75rem;
        display: inline-block;
        font-weight: 600;
        color: var(--color-gray-2);
      }
    }
  }
</style>
