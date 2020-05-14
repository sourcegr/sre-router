[![npm][npm]][npm-url]

# SRE-Router

A dead simple Svelte routing library.

## Install

```bash
npm install --save sre-router
```

## Example usage

```html
<!-- this is app.svelte -->
<script>
    import {Route} from 'sre-router';
    import {Link} from 'sre-router';
</script>

<Route match="products">
    <p>Will show</p>
    <ul>
    
</ul> 
</Route>
```