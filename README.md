# site

Personal portfolio. SvelteKit 2 + Svelte 5 + TypeScript, with a matrix-rain
canvas background that dims and slows when you hover a project card.

## run

```sh
npm install
npm run dev
```

## build

```sh
npm run build
npm run preview
```

## notes

- Hovering the title or bio runs a letter-rotation animation.
- Project cards are data-driven — edit the `projects` array in
  `src/routes/+page.svelte`.
