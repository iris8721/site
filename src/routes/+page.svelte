<script lang="ts">
	import { onMount } from 'svelte';

	console.log("By this art you may contemplate the variation of the 23 letters ඞ");

	const name = "Finn Watt";
	const title = "thousandaire";
	const bio = "aura coder";
	const email = "flexion360@gmail.com";

	const badges: Record<string, { label: string; color: string }> = {
		closed: { label: "closed source", color: "#f85149" },
		open: { label: "open source", color: "#3fb950" },
		wip: { label: "work in progress", color: "#d29922" }
	};

	const tags: Record<string, { label: string; color: string; link?: string }> = {
		rust: { label: "rust", color: "#dea584", link: "https://www.rust-lang.org" },
		svelte: { label: "svelte", color: "#ff3e00", link: "https://svelte.dev" },
		postgres: { label: "postgres", color: "#336791", link: "https://www.postgresql.org" },
		typescript: { label: "typescript", color: "#3178c6", link: "https://www.typescriptlang.org" },
		react: { label: "react", color: "#61dafb", link: "https://react.dev" },
		python: { label: "python", color: "#3776ab", link: "https://www.python.org" },
		pytorch: { label: "pytorch", color: "#ee4c2c", link: "https://pytorch.org" },
		kubernetes: { label: "kubernetes", color: "#326ce5", link: "https://kubernetes.io" },
		java: { label: "java", color: "#b07219", link: "https://www.java.com" },
		node: { label: "node.js", color: "#5fa04e", link: "https://nodejs.org" },
		go: { label: "go", color: "#00add8", link: "https://go.dev" }
	};

	const projects = [
		{
			name: "umbra",
			badge: "open",
			image: "",
			subtitle: "Embeddable scripting language",
			description: "Lua-inspired language built from scratch in Rust: lexer, bytecode VM, garbage collector, and a small C FFI for embedding.",
			tags: ["rust"],
			link: "https://github.com/iris8721/umbra"
		},
		{
			name: "waterloo-craft",
			badge: "open",
			image: "",
			subtitle: "Minecraft server plugin suite",
			description: "Thirteen custom PaperMC plugins — duels, bounties, combat logging, shops — powering a Waterloo community Minecraft server.",
			tags: ["java"],
			link: "https://github.com/iris8721/waterloo-craft"
		},
		{
			name: "deadlock-player-finder",
			badge: "open",
			image: "",
			subtitle: "Deadlock player identification",
			description: "Identifies players in Valve's Deadlock by correlating match fingerprints across game history.",
			tags: ["node"],
			link: "https://github.com/iris8721/deadlock-player-finder"
		},
		{
			name: "grt",
			badge: "open",
			image: "",
			subtitle: "Live transit map",
			description: "Real-time map of GRT buses and the ION LRT in Waterloo Region, served from GTFS schedule and realtime feeds.",
			tags: ["go"],
			link: "https://github.com/iris8721/grt"
		},
		{
			name: "wordle",
			badge: "open",
			image: "",
			subtitle: "Terminal Wordle solver",
			description: "Zero-dependency Rust CLI that narrows the answer list from your guesses and ranks the next move by expected information gain.",
			tags: ["rust"],
			link: "https://github.com/iris8721/wordle"
		}
	];

	let copied = $state(false);
	let copyTimeout: ReturnType<typeof setTimeout>;
	let projectHover = $state(false);
	let titleChars = $state(title.split(''));
	let bioChars = $state(bio.split(''));
	let canvas: HTMLCanvasElement;

	function createTextAnimation(getText: () => string[], setText: (chars: string[]) => void, original: string, interval: number) {
		let timer: ReturnType<typeof setInterval> | null = null;

		const increment = (char: string) => {
			if (!/[a-zA-Z]/.test(char)) return { char, carry: false };
			const base = char === char.toUpperCase() ? 65 : 97;
			const code = char.charCodeAt(0) - base;
			return code === 25
				? { char: String.fromCharCode(base), carry: true }
				: { char: String.fromCharCode(base + code + 1), carry: false };
		};

		const advance = (chars: string[]) => {
			const result = [...chars];
			let carry = true;
			for (let i = result.length - 1; i >= 0 && carry; i--) {
				const { char, carry: c } = increment(result[i]);
				result[i] = char;
				carry = c;
			}
			return result;
		};

		return {
			start: () => {
				if (timer) return;
				timer = setInterval(() => setText(advance(getText())), interval);
			},
			stop: () => {
				if (timer) clearInterval(timer);
				timer = null;
				setText(original.split(''));
			}
		};
	}

	const titleAnim = createTextAnimation(() => titleChars, c => titleChars = c, title, 5);
	const bioAnim = createTextAnimation(() => bioChars, c => bioChars = c, bio, 5);

	function copyEmail() {
		navigator.clipboard.writeText(email);
		copied = true;
		clearTimeout(copyTimeout);
		copyTimeout = setTimeout(() => copied = false, 2000);
	}

	onMount(() => {
		const ctx = canvas.getContext('2d')!;
		const chars = 'abcdefghijklmnopqrstuvwxyz';
		const fontSize = 16;
		const spacing = 80;
		const trailLength = 30;

		let animationId: number;
		let columns: number;
		let drops: number[];
		let offsets: number[];
		let speeds: number[];
		let trails: { char: string; y: number }[][];
		let currentOpacity = 0.12;
		let currentSpeed = 1;

		const initColumns = () => {
			columns = Math.floor(canvas.width / spacing);
			drops = Array.from({ length: columns }, () => Math.random() * (canvas.height / fontSize + 50) - 50);
			offsets = Array.from({ length: columns }, () => (Math.random() - 0.5) * spacing * 0.8);
			speeds = Array.from({ length: columns }, () => 0.05 + Math.random() * 0.1);
			trails = Array.from({ length: columns }, () => []);
		};

		const resize = () => {
			canvas.width = window.innerWidth;
			canvas.height = window.innerHeight;
			initColumns();
		};

		resize();
		window.addEventListener('resize', resize);

		const draw = () => {
			const targetOpacity = projectHover ? 0.02 : 0.12;
			const targetSpeed = projectHover ? 0.3 : 1;
			currentOpacity += (targetOpacity - currentOpacity) * 0.05;
			currentSpeed += (targetSpeed - currentSpeed) * 0.05;

			const effectiveTrailLength = Math.round(trailLength / currentSpeed);

			ctx.clearRect(0, 0, canvas.width, canvas.height);
			ctx.font = `${fontSize}px ui-monospace, monospace`;

			for (let i = 0; i < columns; i++) {
				const x = i * spacing + spacing / 2 + offsets[i];
				const y = drops[i] * fontSize;
				const char = chars[Math.floor(Math.random() * chars.length)];

				trails[i].unshift({ char, y });
				if (trails[i].length > effectiveTrailLength) trails[i].pop();

				for (let j = 0; j < trails[i].length; j++) {
					ctx.fillStyle = `rgba(0, 0, 0, ${currentOpacity * (1 - j / trails[i].length)})`;
					ctx.fillText(trails[i][j].char, x, trails[i][j].y);
				}

				if (y > canvas.height && Math.random() > 0.98) {
					drops[i] = -Math.random() * 20;
					offsets[i] = (Math.random() - 0.5) * spacing * 0.8;
					speeds[i] = 0.05 + Math.random() * 0.1;
					trails[i] = [];
				}
				drops[i] += speeds[i] * currentSpeed;
			}

			animationId = requestAnimationFrame(draw);
		};

		draw();

		return () => {
			cancelAnimationFrame(animationId);
			window.removeEventListener('resize', resize);
		};
	});
</script>

<svelte:head>
	<title>{name} | portfolio</title>
</svelte:head>

<canvas bind:this={canvas} class="bg-canvas"></canvas>

<main>
	<div class="hero">
		<h1>{name}</h1>
		<p class="title" onmouseenter={titleAnim.start} onmouseleave={titleAnim.stop}>{titleChars.join('')}</p>
		<p class="bio" onmouseenter={bioAnim.start} onmouseleave={bioAnim.stop}>{bioChars.join('')}</p>
		<div class="links">
			<a href="https://github.com/synqueue" target="_blank" rel="noopener">GitHub</a>
			<a href="https://www.linkedin.com/in/finn-watt-83a7913ab/" target="_blank" rel="noopener">LinkedIn</a>
			<button onclick={copyEmail}>{copied ? 'Copied!' : 'Email'}</button>
		</div>
	</div>

	<section class="projects">
		<h2>Projects</h2>
		<div class="project-grid">
			{#each projects as project}
				<div class="project-card" role="article" onmouseenter={() => projectHover = true} onmouseleave={() => projectHover = false}>
					<div class="card-header">
						<h3>{project.name}</h3>
						<span class="badge" style="--badge-color: {badges[project.badge].color}">
							{badges[project.badge].label}
						</span>
					</div>
					<div class="card-image">
						{#if project.image}
							<img src={project.image} alt={project.name} />
						{/if}
					</div>
					<h4 class="card-subtitle">{project.subtitle}</h4>
					<p class="card-description">{project.description}</p>
					<div class="card-tags">
						{#each project.tags as tagKey}
							{#if tags[tagKey].link}
								<a href={tags[tagKey].link} class="tag" style="--tag-color: {tags[tagKey].color}" target="_blank" rel="noopener">
									{tags[tagKey].label}
								</a>
							{:else}
								<span class="tag" style="--tag-color: {tags[tagKey].color}">{tags[tagKey].label}</span>
							{/if}
						{/each}
					</div>
					<a href={project.link} class="card-link" target="_blank" rel="noopener">View on GitHub</a>
				</div>
			{/each}
		</div>
	</section>
</main>

<style>
	.bg-canvas {
		position: fixed;
		inset: 0;
		z-index: -1;
		pointer-events: none;
	}

	main {
		min-height: 100vh;
		display: flex;
		flex-direction: column;
		align-items: center;
		font-family: ui-monospace, 'SF Mono', 'Cascadia Code', 'Consolas', monospace;
		user-select: none;
	}

	.hero {
		text-align: center;
		padding: 4rem 2rem;
	}

	h1 {
		font-size: 3rem;
		margin: 0 0 0.5rem;
		font-weight: 700;
	}

	.title, .bio {
		cursor: default;
	}

	.title {
		font-size: 1.25rem;
		color: #666;
		margin: 0 0 1rem;
	}

	.bio {
		font-size: 1.1rem;
		color: #444;
		margin: 0 0 2rem;
	}

	.links {
		display: flex;
		gap: 1.5rem;
		justify-content: center;
	}

	.links a, .links button {
		color: #0070f3;
		text-decoration: none;
		font-weight: 500;
		transition: color 0.2s;
	}

	.links a:hover, .links button:hover {
		color: #0050b3;
	}

	.links button {
		background: none;
		border: none;
		font: inherit;
		cursor: pointer;
		padding: 0;
	}

	.projects {
		width: 100%;
		max-width: 1200px;
		padding: 2rem;
	}

	.projects h2 {
		font-size: 1.5rem;
		margin: 0 0 1.5rem;
		font-weight: 600;
	}

	.project-grid {
		display: grid;
		grid-template-columns: repeat(3, 1fr);
		gap: 1.5rem;
	}

	.project-card {
		background: #fafafa;
		border: 1px solid #e0e0e0;
		padding: 1.25rem;
		color: #333;
		transition: border-color 0.2s;
	}

	.project-card:hover {
		border-color: #0070f3;
	}

	.card-header {
		display: flex;
		justify-content: space-between;
		align-items: center;
		margin-bottom: 1rem;
	}

	.card-header h3 {
		font-size: 1.1rem;
		margin: 0;
		font-weight: 600;
		color: #111;
	}

	.badge {
		font-size: 0.75rem;
		padding: 0.25rem 0.5rem;
		border: 1px solid var(--badge-color);
		color: var(--badge-color);
	}

	.card-image {
		background: #eee;
		height: 180px;
		margin-bottom: 1rem;
		display: flex;
		align-items: center;
		justify-content: center;
		overflow: hidden;
	}

	.card-image img {
		width: 100%;
		height: 100%;
		object-fit: cover;
	}

	.card-subtitle {
		font-size: 0.95rem;
		font-weight: 400;
		margin: 0 0 0.75rem;
		color: #333;
	}

	.card-description {
		font-size: 0.85rem;
		color: #666;
		margin: 0 0 1rem;
		line-height: 1.5;
	}

	.card-tags {
		display: flex;
		flex-wrap: wrap;
		gap: 0.5rem;
		margin-bottom: 1rem;
	}

	.tag {
		font-size: 0.75rem;
		padding: 0.25rem 0.5rem;
		border: 1px solid var(--tag-color);
		color: var(--tag-color);
		text-decoration: none;
		transition: background 0.2s, color 0.2s;
	}

	a.tag:hover {
		background: var(--tag-color);
		color: #fafafa;
	}

	.card-link {
		display: inline-block;
		font-size: 0.85rem;
		padding: 0.5rem 1rem;
		border: 1px solid #58a6ff;
		color: #58a6ff;
		text-decoration: none;
		transition: background 0.2s, color 0.2s;
	}

	.card-link:hover {
		background: #58a6ff;
		color: #0d1117;
	}
</style>
