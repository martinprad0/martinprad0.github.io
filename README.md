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

## GitHub Pages Deployment

### Setting the repository
On a fresh repository
```bash
git init
git remote add origin https://github.com/username/project-name.git
git add .
git commit -m "first commit"
git branch -M main
git push -u origin main
```

If something related to authentication failed on the previous step. Authenticate by SSH remote URL (if you already have your SSH key pasted into your [GitHub Account](https://github.com/settings/keys)).
```bash
git remote set-url origin git@github.com:username/project-name.git
```
(Optional) you can generate this key with
```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
cat ~/.ssh/id_ed25519.pub # Paste this output on New SSH key
```
Now try to push again using
```bash
git push -u origin main
```
If you get
```
! [rejected]        main -> main (fetch first)
error: failed to push some refs to 'github.com:username/project-name.git'
```
try again using `git push -u origin main --force`. The flag `--force` will nuke everything saved on GitHub so be careful.

### GitHub Actions

This is the last step to deploy the site with GitHub Pages. Go to your repository settings