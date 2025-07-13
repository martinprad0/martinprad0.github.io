# SvelteKit deployment to GitHub Pages

This tutorial was made using as reference:

- [Deploy SvelteKit Website to GitHub Pages (Okupter Blog)](https://www.okupter.com/blog/deploy-sveltekit-website-to-github-pages)
- [SvelteKit Adapter Static Documentation](https://svelte.dev/docs/kit/adapter-static)
- [YouTube: Deploy SvelteKit to GitHub Pages](https://www.youtube.com/watch?v=Fw0tbaaGvII)

I'm going to use `pnpm`. If you want to use `npm` follow the guide on the second and third link.

## SvelteKit project creation
If you want to start from zero don't skip this part, otherwise go to the next chapter. Run the following command (For this tutorial I'm assuming your github username is `username` and your project's name is `project-name`)
```bash
npx sv create project-name
cd project-name # Then, go to your project's directory
```
and don't forget to choose `TypeScript` for syntax and `pnpm` for the package manager. This implies that any additional package must be installed using
```bash
pnpm add package-name
```
Then, test if everything is correct by running
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

### GitHub Repository Settings
Create a repository called `project-name` and go to the project settings at `https://github.com/username/project-name/settings`. In the Settings go to:

`Code and automation` > `Pages` > `Build and deployment` > `Source`

and select the option: `GitHub Actions`.

Now create a file `.github/workflows/deploy.yml` on your project with the following contents
```yml
name: Deploy to GitHub Pages

on:
  push:
    branches: ['main']

jobs:
  build_site:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Install pnpm
        uses: pnpm/action-setup@v3
        with:
          version: 8

      - name: Install Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'pnpm'

      - name: Install dependencies
        run: pnpm install

      - name: Build
        run: pnpm build

      - name: Upload Pages artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: build/

  deploy:
    needs: build_site
    runs-on: ubuntu-latest

    permissions:
      pages: write
      id-token: write

    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}

    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

### Pushing the repository
This is common git knowledge, but if you're as clueless as I am, on a fresh repository use
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
try again using `git push -u origin main --force`. 

> **⚠️ Warning:**  
> The `--force` flag will overwrite everything currently saved on GitHub for this branch. Use it with caution to avoid losing important data.


### GitHub Actions

This is the last step to deploy the site with GitHub Pages. Go to your repository settings. Now, you have a Svelte 5 page on
https://username.github.io/project-name/