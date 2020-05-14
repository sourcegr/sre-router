<script context="module">
	import { writable } from 'svelte/store';

	const { subscribe, set } = writable();


	const router = {
		subscribe,
		go: where => event => router._go(where),
		_go: where => {
			let go = true;
			if (is_dirty) {
				go = confirm(dirty_msg);
			}

			if (go) {
				router.clear_dirty();
				if (where.indexOf('/') !== 0) {
					// relative
					where = '/' + router.url.split('/').slice(1, -1).join('/') + '/' + where;
				}
				window.history.pushState(where, '', window.location.origin + where);
				makeStoreValue(where);
			}
		},
		back: e => {
			let go = true;
			if (is_dirty) {
				go = confirm(dirty_msg);
			}
			if (go) {
				history.go(-1);
			}
		},
		set_dirty: (msg = null) => {
			is_dirty = true;
			if (typeof msg != 'string') msg = null;
			dirty_msg = msg || 'Are you sure?';
		},
		clear_dirty: () => {
			is_dirty = false;
		}
	};

	const makeStoreValue = url => {
		const tmp_url = new URL(document.location.protocol + '//' + document.location.host + url);

		let params = null;


		if (tmp_url.search) {
			params = JSON.parse('{"' + decodeURI(tmp_url.search.substring(1)).
				replace(/"/g, '\\"').
				replace(/&/g, '","').
				replace(/=/g,'":"') + '"}')
		}

		let hash = tmp_url.hash || null;

		const parts = tmp_url.pathname.substring(1).split('/');


		set({parts, hash, params});
		router.url = '/' + parts.join('/');
	};

	const now = document.location.pathname + document.location.search + document.location.hash;

	let dirty_msg = null;
	let is_dirty = false;

	window.history.replaceState(now, '');
	makeStoreValue(now);

	window.addEventListener("popstate", function(event) {
		const where = event.state;
		if (where) makeStoreValue(where);
	});

	export { router };
</script>

<script>
	import { getContext, setContext } from 'svelte';

	export let match = '';
	export let exact = null;

	let depth = getContext('depth') || 0;
	let match_result = true;

	if (match[0] == "!" && match.length > 3) {
		match = match.substring(1);
		match_result = false;
	}

	let active, path=[], data = {
		match,
		invert_match: !match_result
	};

	setContext('depth', depth + (exact ? 0 : 1));

	const run = function() {
		active = match_result;
		data.wildcard = null;

		if (
			(exact && depth == path.length) ||
			(path.length == depth && match == '/' || path.length == 1 && path[0] == '' && match == '/') ||
			(path[depth] == match)
		) {
			return;
		}


		if (match == '*' && path.length > depth) {
			data.wildcard = path.slice(depth);
			return;
		}

		active = !match_result;
		data.wildcard = null;
	}


	$: {
		path = $router.parts;
		run();
		let r = $router;
		data.params = r.params;
		data.hash = r.hash;
		data.path = path;
	}

</script>

{#if active}
	<slot router={ data } />
{/if}