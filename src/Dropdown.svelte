<script>
  import { setContext, onDestroy } from 'svelte';
  import classnames from './utils';

  import { createContext } from './DropdownContext';

  let context = createContext();
  setContext('dropdownContext', context);

  let className = '';
  export { className as class };
  export let direction = 'down';
  export let group = false;
  export let isOpen = false;
  export let nav = false;
  export let active = false;
  export let addonType = false;
  export let size = '';
  export let toggle = undefined;
  export let inNavbar = false;
  export let setActiveFromChild = false;
  export let dropup = false;

  const validDirections = ['up', 'down', 'left', 'right'];

  if (validDirections.indexOf(direction) === -1) {
    throw new Error(
      `Invalid direction sent: '${direction}' is not one of 'up', 'down', 'left', 'right'`
    );
  }

  let component;

  $: subItemIsActive = !!(
    setActiveFromChild &&
    component &&
    typeof component.querySelector === 'function' &&
    component.querySelector('.active')
  );

  $: classes = classnames(
    className,
    direction !== 'down' && `drop${direction}`,
    nav && active ? 'active' : false,
    setActiveFromChild && subItemIsActive ? 'active' : false,
    {
      [`input-group-${addonType}`]: addonType,
      'btn-group': group,
      [`btn-group-${size}`]: !!size,
      dropdown: !group && !addonType,
      show: isOpen,
      'nav-item': nav
    }
  );

  $: {
    if (typeof document !== 'undefined') {
      if (isOpen) {
        ['click', 'touchstart', 'keyup'].forEach((event) =>
          document.addEventListener(event, handleDocumentClick, true)
        );
      } else {
        ['click', 'touchstart', 'keyup'].forEach((event) =>
          document.removeEventListener(event, handleDocumentClick, true)
        );
      }
    }
  }

  $: {
    context.update(() => {
      return {
        toggle,
        isOpen,
        direction: direction === 'down' && dropup ? 'up' : direction,
        inNavbar
      };
    });
  }

  function handleDocumentClick(e) {
    if (e && (e.which === 3 || (e.type === 'keyup' && e.which !== 9))) return;

    if (
      component.contains(e.target) &&
      component !== e.target &&
      (e.type !== 'keyup' || e.which === 9)
    ) {
      return;
    }

    toggle(e);
  }
  onDestroy(() => {
    ['click', 'touchstart', 'keyup'].forEach((event) =>
      document.removeEventListener(event, handleDocumentClick, true)
    );
  });
</script>

{#if nav}
  <li {...$$restProps} class={classes} bind:this={component}>
    <slot />
  </li>
{:else}
  <div {...$$restProps} class={classes} bind:this={component}>
    <slot />
  </div>
{/if}
