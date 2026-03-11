<script>
  import { chapters } from '../data/chapters.js';

  let { onChapterSelect } = $props();
</script>

<div class="home">
  <div class="hero">
    <p class="hero-label">Syntax-Trainer</p>
    <h1 class="hero-title">Was hast du heute gelernt?</h1>
    <p class="hero-sub">
      Wähle ein Kapitel und teste dein Verständnis mit zufälligen Aufgaben.
    </p>
  </div>

  <div class="chapter-grid">
    {#each chapters as chapter}
      <button class="chapter-card" onclick={() => onChapterSelect(chapter)}>
        <div class="card-number">Kapitel {chapter.number}</div>
        <div class="card-title">{chapter.title}</div>
        <div class="card-topics">{chapter.topics}</div>
        <div class="card-arrow">→</div>
      </button>
    {/each}

    {#each Array(Math.max(0, 3 - chapters.length)) as _}
      <div class="chapter-card chapter-card--placeholder">
        <div class="card-number">Demnächst</div>
        <div class="card-title">Weiteres Kapitel</div>
        <div class="card-topics">In Vorbereitung</div>
      </div>
    {/each}
  </div>
</div>

<style>
  .home {
    display: flex;
    flex-direction: column;
    gap: 48px;
  }

  .hero {
    padding-top: 16px;
  }

  .hero-label {
    font-family: var(--font-code);
    font-size: 0.78rem;
    color: var(--accent);
    letter-spacing: 0.12em;
    text-transform: uppercase;
    margin-bottom: 14px;
  }

  .hero-title {
    font-size: clamp(1.8rem, 4vw, 2.6rem);
    font-weight: 300;
    color: var(--text);
    letter-spacing: -0.02em;
    line-height: 1.2;
    margin-bottom: 16px;
  }

  .hero-sub {
    font-size: 1rem;
    color: var(--text-muted);
    max-width: 500px;
    line-height: 1.7;
  }

  .chapter-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(260px, 1fr));
    gap: 16px;
  }

  .chapter-card {
    background: var(--surface);
    border: 1px solid var(--border);
    border-radius: var(--radius-lg);
    padding: 24px;
    text-align: left;
    display: flex;
    flex-direction: column;
    gap: 8px;
    position: relative;
    transition: border-color 0.2s, background 0.2s, transform 0.15s;
    overflow: hidden;
  }

  .chapter-card::before {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    height: 2px;
    background: var(--accent);
    transform: scaleX(0);
    transform-origin: left;
    transition: transform 0.2s;
  }

  .chapter-card:hover {
    border-color: var(--accent);
    background: var(--surface-2);
    transform: translateY(-2px);
  }

  .chapter-card:hover::before {
    transform: scaleX(1);
  }

  .chapter-card--placeholder {
    opacity: 0.35;
    cursor: not-allowed;
  }

  .chapter-card--placeholder:hover {
    transform: none;
    border-color: var(--border);
    background: var(--surface);
  }

  .chapter-card--placeholder::before {
    display: none;
  }

  .card-number {
    font-family: var(--font-code);
    font-size: 0.75rem;
    color: var(--accent);
    letter-spacing: 0.08em;
    text-transform: uppercase;
  }

  .card-title {
    font-size: 1rem;
    font-weight: 600;
    color: var(--text);
    line-height: 1.3;
  }

  .card-topics {
    font-size: 0.82rem;
    color: var(--text-muted);
    line-height: 1.5;
    margin-top: 4px;
  }

  .card-arrow {
    font-size: 1.1rem;
    color: var(--accent);
    margin-top: 8px;
    transition: transform 0.2s;
  }

  .chapter-card:hover .card-arrow {
    transform: translateX(4px);
  }
</style>
