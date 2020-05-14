<script>
   import { router } from './Router.svelte';

   export let real = false;
   export let href = '#';
   export let style = null;
   export let target = null;

   const local_go = event => {
      if (!real) {
         event.preventDefault();
         router._go(href);
      }
   }
</script>


<a { href } { style } { target } on:click="{ local_go }"><slot/></a>