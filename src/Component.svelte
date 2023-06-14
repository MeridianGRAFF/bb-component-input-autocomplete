<script>
  import { getContext, onDestroy } from "svelte";
  import Returned from "./Returned.svelte";

  export let field;
  export let label;
  export let width;
  export let height;
  export let onBlur;
  export let validation;
  export let onEnterKey;
  export let placeholder;
  export let defaultValue = null;
  export let dataProvider;
  export let providerField;
  export let maxRows;
  export let beginSearch;

  const formContext = getContext("form");
  const { styleable } = getContext("sdk");
  const component = getContext("component");
  const formStepContext = getContext("form-step");
  const fieldGroupContext = getContext("field-group");

  let fieldApi;
  let fieldState;
  let filteredResults = [];
  let hiLiteIndex = null;
  let searchInput; // use with bind:this to focus element
  let inputValue = "";
  let autocompleteHover = 0
  let autocompleteClicked = 0

  const formApi = formContext?.formApi;
  const labelPos = fieldGroupContext?.labelPosition || "above";

  $: providerResults = dataProvider?.rows;

  $: formStep = formStepContext ? $formStepContext || 1 : 1;

  $: formField = formApi?.registerField(
    field,
    "text",
    defaultValue,
    false,
    validation,
    formStep
  );

  $: unsubscribe = formField?.subscribe((value) => {
    fieldState = value?.fieldState;
    fieldApi = value?.fieldApi;
  });

  $: value = fieldState?.value || defaultValue;

  $: labelClass =
    labelPos === "above" ? "" : `spectrum-FieldLabel--${labelPos}`;

  $: overrideWdith = width ? width + "px" : "auto";
  $: overrideHeight = height ? height + "px" : "auto";

  $: if (!inputValue) {
    filteredResults = [];
    hiLiteIndex = null;
  };

  $: hiLitedResult = filteredResults[hiLiteIndex];

  const clearInput = () => {
    inputValue = "";
  };

  const setInputVal = (resultName) => {
    inputValue = removeBold(resultName);
    filteredResults = [];
    hiLiteIndex = null;
    fieldApi?.setValue(inputValue);
    autocompleteClicked = 1;
    document.querySelector('input.autocomplete-input').focus();
  };

  const filterResults = (e) => {
    if autocompleteClicked === 1 {
      autocompleteClicked = 0;
      clearInput();
      return;
    };
    if (e.target.value) {
      e.target.value = e.target.value.replace(/\s/g, '');
      inputValue = e.target.value;
    };
    let storageArr = []
    filteredResults = []
    if (inputValue && inputValue.length >= beginSearch) {
      storageArr = providerResults.filter(results => {
           return results[providerField].toLowerCase().search(inputValue.toLowerCase()) != -1
      });
      for (let i = 0; i < maxRows && i < storageArr.length; i++) {
        filteredResults = [...filteredResults, makeMatchBold(storageArr[i][providerField])];
      };
    };
  };

  const makeMatchBold = (str) => {
    // replace part of (providerField value === inputValue) with strong tags
    let matched = str.substring(str.toUpperCase().indexOf(inputValue.toUpperCase()), str.toUpperCase().indexOf(inputValue.toUpperCase()) + inputValue.length);
    let makeBold = `<strong>${matched}</strong>`;
    let boldedMatch = str.replace(matched, makeBold);
    return boldedMatch;
  };

  const removeBold = (str) => {
    //replace < and > all characters between
    return str.replace(/<(strong)>/g, "").replace(/<\/(strong)>/g, "");
  };

  const handleInputEnterKey = (e) => {
    value = e.target.value
    if (e.keyCode === 40 && hiLiteIndex <= filteredResults.length-1) {
      e.preventDefault();
      hiLiteIndex === null ? hiLiteIndex = 0 : hiLiteIndex += 1;
      e.target.selectionEnd = value.length + 1;
      e.target.selectionStart = value.length + 1;
    } else if (e.keyCode === 38 && hiLiteIndex !== null) {
      e.preventDefault();
      hiLiteIndex === 0 ? hiLiteIndex = null : hiLiteIndex -= 1;
      e.target.selectionEnd = value.length + 1;
      e.target.selectionStart = value.length + 1;
    } else if (e.keyCode === 13 && value && onEnterKey) {
      if (hiLiteIndex !== null) {
        setInputVal(filteredResults[hiLiteIndex]);
        value = inputValue;
        onEnterKey({ value });
      } else {
        onEnterKey({ value });
        clearInput();
      };
    } else {
      return;
    }
    //console.log(e);
  };

  const handleInputBlur = (e) => {
    value = e.target.value

    if (e.type === "blur" && value && onBlur) {
      onBlur({ value });
    }
    if (autocompleteHover === 0) {
      clearInput();
    };
  };

  const hoverOn = () => {
    autocompleteHover = 1;
  };

  const hoverOff = () => {
    autocompleteHover = 0;
  };

  onDestroy(() => {
    fieldApi?.deregister();
    unsubscribe?.();
  });
</script>

<div class="spectrum-Form--labelsAbove">
  <div class="spectrum-Form-item" use:styleable={$component.styles}>
    {#if !formContext}
      <div class="placeholder">
        Form components need to be wrapped in a form
      </div>
    {:else}
      <label
        class:hidden={!label}
        for={fieldState?.fieldId}
        class={`spectrum-FieldLabel spectrum-FieldLabel--sizeM spectrum-Form-itemLabel ${labelClass}`}
      >
        {label || " "}
      </label>

      <div class="spectrum-Form-itemField">
        <div
          class="spectrum-Textfield"
          style={`width: ${overrideWdith};height:${overrideHeight};`}
        >
          <input
            {value}
            type="text"
            {placeholder}
            inputmode="text"
            on:blur={handleInputBlur}
            on:keydown={handleInputEnterKey}
            on:input={filterResults}
            on:focus={filterResults}
            class="spectrum-Textfield-input autocomplete-input"
          />
        </div>

        <div class="autocomplete" style={`width: ${overrideWdith};`}>
          {#if filteredResults.length > 0}
            <ul id="autocomplete-items-list" on:mouseenter={hoverOn} on:mouseleave={hoverOff}>
              {#each filteredResults as results, i}
                {#if i < maxRows}
                  <Returned itemLabel={results} highlighted={i === hiLiteIndex} on:click={() => setInputVal(results)}/>
                {/if}
              {/each}     
            </ul>
          {/if}
        </div>

        {#if fieldState?.error}
          <div class="error">{fieldState.error}</div>
        {/if}
      </div>
    {/if}
  </div>
</div>

<style>
  .placeholder {
    color: var(--spectrum-global-color-gray-600);
  }

  label {
    white-space: nowrap;
  }

  label.hidden {
    padding: 0;
  }

  .spectrum-Form-itemField {
    position: relative;
    width: 100%;
  }

  .error {
    color: var(--spectrum-semantic-negative-color-default, 
               var(--spectrum-global-color-red-500));
    font-size: var(--spectrum-global-dimension-font-size-75);
    margin-top: var(--spectrum-global-dimension-size-75);
  }

  .spectrum-FieldLabel--right,
  .spectrum-FieldLabel--left {
    padding-right: var(--spectrum-global-dimension-size-200);
  }

  div.autocomplete {
    /*the container must be positioned relative:*/
    position: relative;
  }

  #autocomplete-items-list {
    position: absolute;
    margin: 0;
    padding: 0;
    top: 0;
    width: inherit;
    border: 1px solid #ddd;
    background-color: #ddd;
    z-index: 99;
    max-height: 40em;
    overflow-y: scroll;
    overflow-x: clip;
  }

</style>