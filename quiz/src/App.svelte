<script>
  import './app.css';
  import { chapters } from './data/chapters.js';
  import Home from './routes/Home.svelte';
  import Chapter from './routes/Chapter.svelte';

  let view = $state('home');
  let activeChapter = $state(null);

  // URL-Parameter beim Start auslesen: ?chapter=02
  const params = new URLSearchParams(window.location.search);
  const chapterParam = params.get('chapter');
  if (chapterParam) {
    const found = chapters.find(c => String(c.number).padStart(2, '0') === chapterParam);
    if (found) {
      activeChapter = found;
      view = 'chapter';
    }
  }

  function openChapter(chapter) {
    activeChapter = chapter;
    view = 'chapter';
    // URL aktualisieren ohne Seitenneuladen
    const url = new URL(window.location);
    url.searchParams.set('chapter', String(chapter.number).padStart(2, '0'));
    window.history.pushState({}, '', url);
  }

  function goHome() {
    view = 'home';
    activeChapter = null;
    // URL-Parameter entfernen
    const url = new URL(window.location);
    url.searchParams.delete('chapter');
    window.history.pushState({}, '', url);
  }
</script>

<div class="app">
  <header>
    <div class="header-inner">
      <div class="logo">
        <span class="logo-mark">▸</span>
        <span class="logo-text">Python im Maschinenbau</span>
      </div>
      {#if view === 'chapter'}
        <button class="back-btn" onclick={goHome}>
          ← Kapitelübersicht
        </button>
      {/if}
    </div>
  </header>

  <main>
    {#if view === 'home'}
      <Home onChapterSelect={openChapter} />
    {:else}
      <Chapter chapter={activeChapter} onBack={goHome} />
    {/if}
  </main>
</div>

<style>
  .app {
    display: flex;
    flex-direction: column;
    min-height: 100vh;
  }

  header {
    border-bottom: 1px solid var(--border);
    background: var(--surface);
    position: sticky;
    top: 0;
    z-index: 10;
  }

  .header-inner {
    max-width: 860px;
    margin: 0 auto;
    padding: 0 24px;
    height: 56px;
    display: flex;
    align-items: center;
    justify-content: space-between;
  }

  .logo {
    display: flex;
    align-items: center;
    gap: 10px;
  }

  .logo-mark {
    color: var(--accent);
    font-size: 1.1rem;
    font-family: var(--font-code);
  }

  .logo-text {
    font-size: 0.9rem;
    font-weight: 500;
    color: var(--text-muted);
    letter-spacing: 0.01em;
  }

  .back-btn {
    background: none;
    border: 1px solid var(--border);
    color: var(--text-muted);
    font-size: 0.85rem;
    padding: 6px 14px;
    border-radius: var(--radius);
    transition: all 0.15s;
  }

  .back-btn:hover {
    border-color: var(--accent);
    color: var(--accent);
  }

  main {
    flex: 1;
    max-width: 860px;
    margin: 0 auto;
    width: 100%;
    padding: 40px 24px;
  }
</style>
