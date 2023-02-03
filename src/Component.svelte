<svelte:head>
  <script src="https://www.google.com/recaptcha/api.js"></script>
  <script>
    function recaptcha_callback()
    {
      document.getElementById("sillek_code").value = grecaptcha.getResponse();
      document.getElementById("sillek_code").dispatchEvent(new Event("change"));
    }
  </script>
</svelte:head>
<script>
  import { getContext } from "svelte"
  import { onDestroy } from "svelte";

  export let field;
  export let sillek_code;
  export let label;
  export let dataSitekey;
  export let validation;
  
  let fieldApi;
  let fieldState;

  const { styleable } = getContext("sdk");
  const component = getContext("component");
  const formContext = getContext("form");
  const formStepContext = getContext("form-step");
  const fieldGroupContext = getContext("field-group");

  const formApi = formContext?.formApi
  const labelPos = fieldGroupContext?.labelPosition || "above";

  $: formStep = formStepContext ? $formStepContext || 1 : 1
  $: formField = formApi?.registerField(field, "string", "0", false, validation, formStep)
  $: unsubscribe = formField?.subscribe((value) => {
    fieldState = value?.fieldState;
    fieldApi = value?.fieldApi;
  });

  $: labelClass =
    labelPos === "above" ? "" : `spectrum-FieldLabel--${labelPos}`;
  
  onDestroy(() => {
    fieldApi?.deregister();
    unsubscribe?.();
});



</script>
<div class="spectrum-Form-item" use:styleable={$component.styles}>
{#if !formContext}
<div class="placeholder">Form components need to be wrapped in a form</div>
{:else}
  <label
    class:hidden={!label}
    for={fieldState?.fieldId}
    class={`spectrum-FieldLabel spectrum-FieldLabel--sizeM spectrum-Form-itemLabel ${labelClass}`}
  >
    {label || " "}
  </label>
  <div class="g-recaptcha" data-sitekey="{dataSitekey}" data-callback="recaptcha_callback" ></div>
  <input bind:this={sillek_code} id="sillek_code" on:change={() => fieldApi?.setValue(sillek_code.value)} />
  {#if !field}
    <div class="error">Please select a field</div>
  {/if}
  {#if fieldState?.error}
    <div class="error">{fieldState.error}</div>
  {/if}
{/if}
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
    color: var(
      --spectrum-semantic-negative-color-default,
      var(--spectrum-global-color-red-500)
    );
    font-size: var(--spectrum-global-dimension-font-size-75);
    margin-top: var(--spectrum-global-dimension-size-75);
  }
  .spectrum-FieldLabel--right,
  .spectrum-FieldLabel--left {
    padding-right: var(--spectrum-global-dimension-size-200);
  }
</style>