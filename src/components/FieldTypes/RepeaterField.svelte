<script>
  import { find as _find } from 'lodash-es'
  import pluralize from '../../libraries/pluralize';
  import { fade } from 'svelte/transition';
  import { createEventDispatcher } from 'svelte';
  const dispatch = createEventDispatcher();

  import { createUniqueID } from '../../utilities';
  import { Card } from '../misc';
  import fieldTypes from '../../stores/app/fieldTypes';

  export let field;

  function addRepeaterItem() {
    const keys = field.fields.map((f) => f.key);
    fieldValues = [...fieldValues, createSubfield()];
    onInput();
  }

  function removeRepeaterItem(itemIndex) {
    fieldValues = fieldValues.filter((_, i) => i !== itemIndex);
    onInput();
  }

  function moveRepeaterItem(indexOfItem, direction) {
    const item = fieldValues[indexOfItem];
    const withoutItem = fieldValues.filter((_, i) => i !== indexOfItem);
    if (direction === 'up') {
      fieldValues = [
        ...withoutItem.slice(0, indexOfItem - 1),
        item,
        ...withoutItem.slice(indexOfItem - 1),
      ];
    } else if (direction === 'down') {
      fieldValues = [
        ...withoutItem.slice(0, indexOfItem + 1),
        item,
        ...withoutItem.slice(indexOfItem + 1),
      ];
    } else {
      console.error('Direction must be up or down');
    }
  }

  function createSubfield() {
    return field.fields.map((subfield) => ({
      ...subfield,
      id: createUniqueID(),
      value: '',
    }));
  }

  let fieldValues = Array.isArray(field.value)
    ? field.value.map((value) => [
        ...field.fields.map((subfield) => ({
          ...subfield,
          value: value[subfield.key],
        })),
      ])
    : [];

  function onInput() {
    field.value = fieldValues.map((fieldValue) =>
      fieldValue.reduce(
        (obj, item) => Object.assign(obj, { [item.key]: item.value }),
        {}
      )
    );
    dispatch('input');
  }

</script>

<Card variants="pb-4" id="repeater-{field.key}">
  <header class="">{field.label}</header>
  <div class="fields">
    {#each fieldValues as fieldValue, i}
      <div
        class="repeater-item"
        id="repeater-{field.key}-{i}"
        in:fade={{ duration: 100 }}>
        <div class="item-options">
          {#if i !== 0}
            <button
              title="Move {field.label} up"
              on:click={() => moveRepeaterItem(i, 'up')}>
              <i class="fas fa-arrow-up" />
            </button>
          {/if}
          {#if i !== fieldValues.length - 1}
            <button
              title="Move {field.label} down"
              on:click={() => moveRepeaterItem(i, 'down')}>
              <i class="fas fa-arrow-down" />
            </button>
          {/if}
          <button
            title="Delete {field.label} item"
            on:click={() => removeRepeaterItem(i)}>
            <i class="fas fa-trash" />
          </button>
        </div>
        {#each fieldValue as subfield}
          <div
            class="repeater-item-field"
            id="repeater-{field.key}-{i}-{subfield.key}">
            <svelte:component
              this={_find($fieldTypes, ['id', subfield.type]).component}
              field={subfield}
              on:input={onInput} />
          </div>
        {/each}
      </div>
    {/each}
    <button class="field-button" on:click={() => addRepeaterItem()}>
      <i class="fas fa-plus mr-1" />
      <span>Add {pluralize.singular(field.label)}</span>
    </button>
  </div>
</Card>

<style lang="postcss">
  header {
    width: 100%;
    padding: 0.25rem 0;
    font-weight: 700;
    font-size: var(--font-size-2);
  }

  .fields {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 1rem;
    grid-row-gap: 2rem;
  }
  .repeater-item {
    flex: 1;
    padding: 1rem;
    background: var(--color-gray-9);
    display: flex;
    flex-direction: column;
    position: relative;
    border: 1px solid var(--color-gray-8);
    border-radius: 1px;
    min-width: 10rem;
    padding-top: 2.5rem;

    &:last-of-type {
      margin-bottom: 0;
    }

    .item-options {
      position: absolute;
      top: 0;
      right: 0;
      left: 0;
      padding: 0.25rem 0.5rem;
      color: var(--color-gray-2);
      z-index: 10;
      border-radius: 1px;
      border-bottom: 1px solid var(--color-gray-8);
      gap: 9px;
      display: flex;
      justify-content: flex-end;

      button {
        &:focus {
          outline: 0;
        }
        &:hover {
          color: var(--primo-color-primored);
        }
        &:last-child {
          margin-left: 0.5rem;
          color: var(--color-gray-5);

          &:hover {
            color: var(--primo-color-primored);
          }
        }
      }
    }
  }
  .repeater-item-field {
    margin-bottom: 0.5rem;
  }
  .repeater-item-field:not(:first-child) {
    padding-top: 0;
  }
  .field-button {
    width: 100%;
    background: var(--color-gray-8);
    border: 1px solid var(--primo-color-primored);
    color: var(--color-gray-3);
    padding: 0.5rem 0;
    border-radius: 1px;
    transition: background 0.1s, color 0.1s;

    &:hover {
      background: var(--primo-color-primored);
    }

    /* &[disabled] {
      background: var(--color-gray-5);
      cursor: not-allowed;
    } */

    i {
      margin-right: 0.5rem;
    }
  }

</style>
