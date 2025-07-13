# SvelteKit deployment to GitHub Pages

The tutorials followed in:

- [Deploy SvelteKit Website to GitHub Pages (Okupter Blog)](https://www.okupter.com/blog/deploy-sveltekit-website-to-github-pages)
- [SvelteKit Adapter Static Documentation](https://svelte.dev/docs/kit/adapter-static)
- [YouTube: Deploy SvelteKit to GitHub Pages](https://www.youtube.com/watch?v=Fw0tbaaGvII)

## SvelteKit project creation
If you want to start from zero don't skip this part, otherwise go to the next chapter. Run the following command (change `testing` with your project's name)
```bash
npx sv create testing
cd testing # Then, go to your project's directory
```
and don't forget to choose `TypeScript` for syntax and `pnpm` for the package manager. This implies that any additional package must be installed using
```bash
pnpm add package-name
```
Then, test if everything is correct by run
```bash
pnpm install
pnpm dev
```

## Deployment using pnpm

In order to host in GitHub Pages we require to use a static adapter package for SvelteKit. This can be installed using
```bash
pnpm add -D @sveltejs/adapter-static
```

### File changes

Using the clean project creation I have the following files:
- For `svelte.config.js`
```js
import adapter from '@sveltejs/adapter-static';
import { vitePreprocess } from '@sveltejs/vite-plugin-svelte';

/** @type {import('@sveltejs/kit').Config} */
const config = {
	// Consult https://svelte.dev/docs/kit/integrations
	// for more information about preprocessors
	preprocess: vitePreprocess(),

	kit: {
		adapter: adapter({
			fallback: '404.html'
		}),
		paths: {
			base: process.argv.includes('dev') ? '' : process.env.BASE_PATH
		}
	}
};

export default config;
```
- I have in `src/routes/+layout.ts`
```ts
export const prerender = true;
```
- I created an empty file `static/.nojekyll`

### 