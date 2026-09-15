# site

Personal portfolio. SvelteKit 2 + Svelte 5 + TypeScript, with a matrix-rain
canvas background that dims and slows when you hover the project grid.

## run

```sh
pnpm install
pnpm dev
```

## build

```sh
pnpm build
pnpm preview
```

## notes

- Hovering the title or bio runs a letter-rotation animation.
- Project cards are data-driven — edit the `projects` array in
  `src/routes/+page.svelte`.
