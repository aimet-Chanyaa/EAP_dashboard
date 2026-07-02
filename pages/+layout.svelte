<script>
  import { EvidenceDefaultLayout } from '@evidence-dev/core-components';
  import { showQueries } from '@evidence-dev/component-utilities/stores';
  import { onMount } from 'svelte';
  export let data;

  // Force "Show Queries" off for everyone, every load — overrides both the
  // dev-mode default (true) and any prior localStorage choice in this browser.
  // This is what removes the "▸ query_name / N records" disclosure under each
  // SQL block; it doesn't touch the "Hide/Show Queries" item in the kebab (⋮)
  // menu, which is still there but starts unchecked again on every reload.
  onMount(() => {
    showQueries.set(false);
  });

  // ----- Sidebar navigation: single scrolling page, 6 anchored sections -----
  // (mirrors the original HTML mock's 6-section anchor nav, not separate routes)
  const navItems = [
    { ix: '01', label: 'Reach & Adoption', id: 's-reach' },
    { ix: '02', label: 'Demographics', id: 's-demo' },
    { ix: '03', label: 'Engagement', id: 's-eng' },
    { ix: '04', label: 'Wellbeing', id: 's-health' },
    { ix: '05', label: 'Deep Insight', id: 's-insight' },
    { ix: '06', label: 'Action Plan', id: 's-action' }
  ];

  let activeId = navItems[0].id;

  onMount(() => {
    const sections = navItems
      .map((item) => document.getElementById(item.id))
      .filter(Boolean);
    if (sections.length === 0) return;

    const observer = new IntersectionObserver(
      (entries) => {
        entries.forEach((entry) => {
          if (entry.isIntersecting) {
            activeId = entry.target.id;
          }
        });
      },
      { rootMargin: '-100px 0px -70% 0px', threshold: 0 }
    );
    sections.forEach((s) => observer.observe(s));
    return () => observer.disconnect();
  });
</script>

<!--
  ====================================================================
  AIMET / Emmind — Evidence.dev Custom Layout
  Replicates the navy/blue "EAP Wellness Dashboard" design system:
  fixed left sidebar nav, sticky topbar, KPI-card grid, rounded cards.
  Edit THIS FILE ONCE — every .md page in /pages automatically
  inherits the look. Page authors only write SQL + chart components.
  ====================================================================
-->

<svelte:head>
  <link rel="preconnect" href="https://fonts.googleapis.com" />
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
  <link
    href="https://fonts.googleapis.com/css2?family=Kanit:wght@300;400;500;600;700&family=Montserrat:wght@600;700;800&display=swap"
    rel="stylesheet"
  />
</svelte:head>

<div class="aimet-shell">
  <!-- ===== Sidebar ===== -->
  <aside class="sidebar">
    <div class="brand">
      <div class="logo"><span class="ai">AI</span><span class="met">MET</span></div>
      <div class="tag">Emmind · EAP Wellness</div>
    </div>
    <nav class="navlist">
      {#each navItems as item}
        <a href={`#${item.id}`} class="navbtn" class:active={activeId === item.id}>
          <span class="ix">{item.ix}</span>{item.label}
        </a>
      {/each}
    </nav>
    <div class="foot">
      <b>ข้อมูลรวมแบบไม่ระบุตัวตน</b><br />
      รายงานระดับองค์กรเท่านั้น<br />
      © 2026 AIMET
    </div>
  </aside>

  <!-- ===== Main content (Evidence renders markdown pages here) ===== -->
  <div class="main">
    <EvidenceDefaultLayout {data} fullWidth={true} hideSidebar={true} hideHeader={true} hideTOC={true}>
      <slot slot="content" />
    </EvidenceDefaultLayout>
  </div>
</div>

<style>
  :root {
    --navy: #002147;
    --navy2: #013a73;
    --blue: #0099ff;
    --blue-d: #0a6db0;
    --blue-l: #4db8ff;
    --red: #eb1e4e;
    --red-m: #ee5c79;
    --red-l: #f59ca9;
    --ink: #0e2138;
    --muted: #5b6b82;
    --bg: #eef3f9;
    --card: #ffffff;
    --line: #e2e9f2;
    --shadow: 0 1px 2px rgba(0, 33, 71, 0.05), 0 10px 28px rgba(0, 33, 71, 0.06);
    --sidebar-w: 236px;
  }

  :global(html) {
    scroll-behavior: smooth;
  }

  :global(body) {
    background: var(--bg) !important;
    font-family: 'Kanit', sans-serif !important;
    color: var(--ink) !important;
    font-size: 14.5px !important;
    line-height: 1.5 !important;
    font-weight: 300 !important;
  }

  /* ===== Grid alignment fix =====
     Evidence's <Grid> component sizes itself purely with Tailwind classes
     (grid-cols-1 sm:grid-cols-2, gap-4) — same systemic issue as the table
     icons/filter buttons: those utility classes aren't reliably applying
     in this build, so 2-up grids can collapse to one column with no gap
     instead of sitting side-by-side like the mockup's .g2 cards. Force the
     layout with plain CSS so columns/gaps render regardless. */
  :global(.main .grid) {
    display: grid !important;
    grid-template-columns: repeat(2, 1fr) !important;
    gap: 18px !important;
  }

  /* Anchor targets for the single-page section nav */
  :global(.main [id^='s-']) {
    scroll-margin-top: 24px;
  }

  .aimet-shell {
    display: flex;
    min-height: 100vh;
  }

  /* ===== Sidebar ===== */
  .sidebar {
    position: fixed;
    top: 0;
    left: 0;
    width: var(--sidebar-w);
    height: 100vh;
    background: linear-gradient(180deg, #002147, #012d56);
    color: #fff;
    display: flex;
    flex-direction: column;
    padding: 24px 18px;
    z-index: 40;
    box-sizing: border-box;
  }

  .brand {
    display: flex;
    flex-direction: column;
    gap: 2px;
    margin-bottom: 6px;
  }

  .logo {
    font-family: 'Montserrat', sans-serif;
    font-weight: 800;
    font-size: 27px;
    letter-spacing: 1px;
    line-height: 1;
  }
  .logo .ai {
    color: var(--blue);
  }
  .logo .met {
    color: #fff;
  }

  .brand .tag {
    font-size: 11px;
    color: #9fb8d4;
    font-weight: 400;
    letter-spacing: 0.5px;
    margin-top: 4px;
  }

  .navlist {
    display: flex;
    flex-direction: column;
    gap: 4px;
    margin-top: 26px;
  }

  .navbtn {
    display: flex;
    align-items: center;
    gap: 11px;
    font-family: 'Kanit', sans-serif;
    font-size: 13.5px;
    font-weight: 400;
    color: #cdddf0;
    text-decoration: none;
    padding: 10px 12px;
    border-radius: 10px;
    transition: 0.16s;
    width: 100%;
    box-sizing: border-box;
  }
  .navbtn .ix {
    font-family: 'Montserrat', sans-serif;
    font-weight: 700;
    font-size: 11px;
    color: #5e87b5;
    width: 16px;
  }
  .navbtn:hover {
    background: rgba(255, 255, 255, 0.07);
    color: #fff;
  }
  .navbtn.active {
    background: var(--blue);
    color: #fff;
    font-weight: 500;
  }
  .navbtn.active .ix {
    color: #fff;
  }

  .sidebar .foot {
    margin-top: auto;
    font-size: 10.5px;
    color: #80a0c4;
    line-height: 1.6;
    border-top: 1px solid rgba(255, 255, 255, 0.12);
    padding-top: 14px;
  }
  .sidebar .foot b {
    color: #bcd2ea;
    font-weight: 500;
  }

  /* ===== Main ===== */
  .main {
    margin-left: var(--sidebar-w);
    width: calc(100% - var(--sidebar-w));
    min-height: 100vh;
  }

  /* ===== Override Evidence default typography to match Kanit / navy theme ===== */
  :global(.main h1.markdown),
  :global(.main h1.title) {
    font-family: 'Montserrat', sans-serif !important;
    color: var(--navy) !important;
    font-weight: 700 !important;
  }
  :global(.main h2.markdown) {
    font-size: 20px !important;
    font-weight: 600 !important;
    color: var(--navy) !important;
    margin: 28px 0 2px !important;
    scroll-margin-top: 80px;
  }
  :global(.main h3.markdown) {
    color: var(--navy) !important;
    font-weight: 600 !important;
    font-size: 18px !important;
    margin: 0 0 2px !important;
  }
  :global(.main h4.markdown) {
    color: var(--navy) !important;
    font-weight: 600 !important;
    font-size: 16px !important;
  }
  :global(.main p.markdown) {
    color: var(--muted) !important;
    font-weight: 300 !important;
    font-size: 14px !important;
    margin: 0 0 12px !important;
  }

  /* Evidence wraps auto-generated heading anchors in <a class="markdown">, which
     the framework's own `article.markdown a.markdown` rule forces blue+underline
     via !important. Beat it with a more specific selector (also !important). */
  :global(.main h1.markdown a.markdown),
  :global(.main h2.markdown a.markdown),
  :global(.main h3.markdown a.markdown),
  :global(.main h4.markdown a.markdown) {
    color: inherit !important;
    text-decoration: none !important;
    font-weight: inherit !important;
  }

  /* ===== Card-style wrapper for tables / charts ===== */
  :global(.main .table-container),
  :global(.main .chart-container) {
    background: var(--card) !important;
    border-radius: 16px !important;
    border: 1px solid var(--line) !important;
    box-shadow: var(--shadow) !important;
    padding: 18px 20px !important;
  }

  /* ===== KPI cards (raw HTML wrapper used around <BigValue> in pages/index.md) ===== */
  :global(.main .kpis) {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(190px, 1fr));
    gap: 16px;
    margin: 14px 0 20px;
  }
  :global(.main .kpi) {
    background: var(--card);
    border-radius: 15px;
    border: 1px solid var(--line);
    box-shadow: var(--shadow);
    padding: 17px 18px 15px;
    position: relative;
    overflow: hidden;
    display: flex;
    flex-direction: column;
    justify-content: center;
    min-height: 120px;
  }
  :global(.main .kpi::before) {
    content: '';
    position: absolute;
    left: 0;
    top: 0;
    bottom: 0;
    width: 4px;
    background: var(--blue);
  }
  :global(.main .kpi.k-red::before) {
    background: var(--red);
  }
  :global(.main .kpi .lab) {
    font-size: 16px;
    color: var(--muted);
    font-weight: 400;
  }
  :global(.main .kpi) {
    font-family: 'Kanit', sans-serif;
  }
  /* BigValue ships almost no styling of its own beyond Tailwind utility
     classes (title = <p class="text-sm">, value = <div class="text-xl">).
     Because each card is written as "🤝 <BigValue .../>", markdown wraps the
     emoji + component in a <p>, so ".kpi > div > div" child-chain selectors
     never match. Target BigValue's own classes with descendant selectors. */
  :global(.main .kpi .inline-block) {
    display: block !important;
    width: 100% !important;
    min-width: 0 !important;
    max-width: none !important;
    margin: 0 !important;
    padding: 0 !important;
  }
  :global(.main .kpi p.text-sm) {
    font-size: 16px !important;
    font-weight: 400 !important;
    color: var(--muted) !important;
    margin: 0 0 4px !important;
    line-height: 1.3 !important;
  }
  :global(.main .kpi .text-xl) {
    font-size: 64px !important;
    font-weight: 700 !important;
    font-family: 'Kanit', sans-serif !important;
    color: var(--navy) !important;
    line-height: 1 !important;
    margin: 10px 0 2px !important;
  }
  :global(.main .kpi .text-xl *) {
    font-size: inherit !important;
    font-weight: inherit !important;
    color: inherit !important;
  }
  :global(.main .kpi.k-red .text-xl) {
    color: var(--red) !important;
  }
  @media (max-width: 1080px) {
    :global(.main .kpis) {
      grid-template-columns: 1fr;
    }
  }

  /* ===== Strip Evidence's built-in table chrome (sort carets, pagination,
     download/fullscreen buttons). These size themselves with Tailwind
     arbitrary-value classes (e.g. w-[10px]); in this project those utility
     classes aren't taking effect, so the icons render at native/huge SVG
     size instead. Rather than chase the Tailwind build issue, just remove
     this chrome — it's not needed on a static internal report anyway. */
  :global(.main .table-container th svg),
  :global(.main .table-container .table-footer),
  :global(.main .table-container .pagination) {
    display: none !important;
  }
  :global(.main .table-container svg),
  :global(.main .chart-container svg) {
    max-width: 16px !important;
    max-height: 16px !important;
  }

  /* ===== Action Plan cards (raw HTML used in pages/index.md) ===== */
  :global(.main .actions) {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 16px;
    margin: 14px 0 6px;
  }
  :global(.main .action) {
    background: var(--card);
    border-radius: 15px;
    border: 1px solid var(--line);
    box-shadow: var(--shadow);
    padding: 18px;
    display: flex;
    flex-direction: column;
  }
  :global(.main .action .tag) {
    align-self: flex-start;
    font-size: 11px;
    font-weight: 500;
    padding: 4px 11px;
    border-radius: 8px;
    margin-bottom: 10px;
  }
  :global(.main .t-now) {
    background: #fde7ec;
    color: #c8123c;
  }
  :global(.main .t-soon) {
    background: #e3f3ff;
    color: var(--blue-d);
  }
  :global(.main .t-plan) {
    background: #e9eef5;
    color: var(--navy);
  }
  :global(.main .action h4) {
    margin: 0 0 6px;
    font-size: 14.5px;
    font-weight: 600;
    color: var(--navy);
  }
  :global(.main .action p) {
    margin: 0 0 11px;
    font-size: 12.5px;
    color: #46506b;
    font-weight: 300;
  }
  :global(.main .action .why) {
    font-size: 11.5px;
    color: var(--muted);
    background: #f5f8fc;
    border-radius: 9px;
    padding: 8px 11px;
    margin-top: auto;
    font-weight: 300;
  }
  :global(.main .action .why b) {
    color: var(--navy);
    font-weight: 500;
  }

  @media (max-width: 1080px) {
    .sidebar {
      position: static;
      width: 100%;
      height: auto;
      flex-direction: row;
      flex-wrap: wrap;
      align-items: center;
      padding: 14px 18px;
    }
    .navlist {
      flex-direction: row;
      flex-wrap: wrap;
      margin-top: 0;
      margin-left: auto;
      gap: 4px;
    }
    .navbtn .ix {
      display: none;
    }
    .sidebar .foot {
      display: none;
    }
    .main {
      margin-left: 0;
      width: 100%;
    }
    :global(.main .actions),
    :global(.main .grid) {
      grid-template-columns: 1fr;
    }
  }
</style>
