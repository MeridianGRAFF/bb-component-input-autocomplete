<script>
  export let itemLabel;
	export let highlighted;
</script>

<style>
li.autocomplete-items {
	list-style: none;
  border-bottom: 1px solid #d4d4d4;
  border-color: var(--spectrum-textfield-m-border-color, var(--spectrum-alias-border-color));
  z-index: 99;
  /*position the autocomplete items to be the same width as the container:*/
  top: 100%;
  left: 0;
  right: 0;
	padding: 10px;
  cursor: pointer;
  background-color: var(--spectrum-textfield-m-background-color, var(--spectrum-global-color-gray-50));
  font-family: var(--spectrum-textfield-text-font-family);
  font-size: var(--spectrum-textfield-text-size);
  line-height: var(--spectrum-textfield-text-line-height);
}

li.autocomplete-items:hover {
  /*when hovering an item:*/
  background-color: var(--primaryColorHover);
	color: white;
}

li.autocomplete-items:active {
  /*when navigating through the items using the arrow keys:*/
  background-color: var(--primaryColorHover) !important;
  color: #ffffff;
}	
	
.autocomplete-active {
  /*when navigating through the items using the arrow keys:*/
  background-color: var(--primaryColorHover) !important;
  color: #ffffff;
}
</style>

<li class="autocomplete-items" class:autocomplete-active={highlighted} on:click>{@html itemLabel}</li>