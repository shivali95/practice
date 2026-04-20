# Returning User Two-Tab Experience Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a persistent bottom nav (🌅 Aaj / ✨ Kundali) to `astro-home-daily-v2.html` so returning users can switch between today's daily content and their permanent kundali universe with a cinematic room-entry transition.

**Architecture:** All changes live in a single file (`astro-home-daily-v2.html`). The Kundali tab's components are ported in from `astro-home.html` — key data constants, overlays, and feed components. Tab state lives in the App component; the bottom nav is a small stateless component. The cinematic transition is a CSS-animated overlay that plays when switching TO the Kundali tab.

**Tech Stack:** React 18 (CDN), Babel standalone, vanilla CSS-in-JS, ElevenLabs TTS (same pattern as existing code), no new dependencies.

---

## File Modified
- `astro-home-daily-v2.html` — all changes here, no new files

---

### Task 1: Add Bottom Nav + Tab State

**Files:**
- Modify: `astro-home-daily-v2.html` — App component + new BottomNav component

- [ ] **Step 1: Add `activeTab` state and `kundaliOverlay` state to App**

Find the App component (~line 2187) and add state:
```jsx
const [activeTab, setActiveTab] = useState('aaj'); // 'aaj' | 'kundali'
const [kundaliSection, setKundaliSection] = useState(null); // deep-link target
const [kundaliOverlay, setKundaliOverlay] = useState(null); // planet/story overlays in kundali tab
```

- [ ] **Step 2: Add BottomNav component** (add just before the App component)

```jsx
const BottomNav = ({ active, onChange }) => (
  <div style={{
    position: 'fixed', bottom: 0, left: '50%', transform: 'translateX(-50%)',
    width: 390, maxWidth: '100vw',
    display: 'flex', alignItems: 'center',
    background: 'rgba(10,10,26,0.92)',
    backdropFilter: 'blur(20px)',
    borderTop: '1px solid rgba(255,255,255,0.07)',
    zIndex: 200,
    padding: '0 0 env(safe-area-inset-bottom,0)',
  }}>
    {[
      { id: 'aaj', icon: '🌅', label: 'Aaj' },
      { id: 'kundali', icon: '✨', label: 'Kundali' },
    ].map(tab => (
      <div key={tab.id} onClick={() => onChange(tab.id)} style={{
        flex: 1, display: 'flex', flexDirection: 'column', alignItems: 'center',
        padding: '10px 0 8px', cursor: 'pointer',
        opacity: active === tab.id ? 1 : 0.4,
        transition: 'opacity 200ms ease',
      }}>
        <div style={{ fontSize: 22 }}>{tab.icon}</div>
        <div style={{
          fontSize: 10, fontWeight: active === tab.id ? 700 : 400,
          color: active === tab.id ? '#4ecdc4' : 'rgba(255,255,255,0.6)',
          marginTop: 3, letterSpacing: 0.5,
        }}>{tab.label}</div>
        {active === tab.id && (
          <div style={{
            width: 20, height: 2, borderRadius: 1,
            background: '#4ecdc4', marginTop: 3,
          }} />
        )}
      </div>
    ))}
  </div>
);
```

- [ ] **Step 3: Render BottomNav in App and add bottom padding to scroll-feed**

Inside the App return, just before the closing `</div>` of `.phone-frame`:
```jsx
<BottomNav active={activeTab} onChange={(tab) => setActiveTab(tab)} />
```

Add `paddingBottom: 64` to the Aaj scroll-feed div's inline style (it's an inline style on the div, not a CSS class — update the existing inline style on the scroll div around line 2215).

- [ ] **Step 4: Copy to preview and verify nav renders**
```bash
cp /Users/shivali.wason/Documents/devotion/astro-home-daily-v2.html /tmp/devotion_preview/astro-home-daily-v2.html
```
Open `http://localhost:8080/astro-home-daily-v2.html` — bottom nav with 🌅 Aaj and ✨ Kundali should be visible. Tapping does nothing yet (no Kundali content).

- [ ] **Step 5: Commit**
```bash
git add astro-home-daily-v2.html
git commit -m "feat: add bottom nav + activeTab state to daily page"
```

---

### Task 2: Cinematic Kundali Entry Transition

**Files:**
- Modify: `astro-home-daily-v2.html` — new KundaliEntry component + transition logic in App

- [ ] **Step 1: Add `showKundaliEntry` state to App**
```jsx
const [showKundaliEntry, setShowKundaliEntry] = useState(false);
```

- [ ] **Step 2: Add KundaliEntry overlay component** (add near other entry/overlay components)

```jsx
const KundaliEntry = ({ onComplete }) => {
  const [visible, setVisible] = useState(true);
  useEffect(() => {
    const t = setTimeout(() => {
      setVisible(false);
      setTimeout(onComplete, 400);
    }, 1800);
    return () => clearTimeout(t);
  }, []);

  return (
    <div style={{
      position: 'absolute', inset: 0, zIndex: 150,
      background: 'radial-gradient(ellipse at center, #0d0d2e 0%, #0a0a1a 80%)',
      display: 'flex', flexDirection: 'column', alignItems: 'center', justifyContent: 'center',
      opacity: visible ? 1 : 0,
      transition: 'opacity 400ms ease',
      pointerEvents: visible ? 'all' : 'none',
    }}>
      <Starfield count={80} />
      <div style={{
        fontSize: 52,
        animation: 'cosmicEntry 800ms ease-out both',
      }}>✨</div>
      <div style={{
        marginTop: 20, fontSize: 18, fontWeight: 300,
        color: 'rgba(255,255,255,0.9)', letterSpacing: 1,
        animation: 'fadeInUp 600ms ease-out 400ms both',
      }}>Apka Kundali Jagat</div>
      <div style={{
        marginTop: 8, fontSize: 12, fontWeight: 400,
        color: 'rgba(255,255,255,0.4)',
        animation: 'fadeInUp 600ms ease-out 600ms both',
      }}>Aapki puri cosmic identity</div>
    </div>
  );
};
```

- [ ] **Step 3: Wire tab change to trigger entry animation**

Update the `onChange` in BottomNav render:
```jsx
<BottomNav active={activeTab} onChange={(tab) => {
  if (tab === 'kundali' && activeTab !== 'kundali') {
    setShowKundaliEntry(true);
    setActiveTab('kundali');
  } else {
    setActiveTab(tab);
  }
}} />
```

Render entry inside the screen div:
```jsx
{showKundaliEntry && (
  <KundaliEntry onComplete={() => setShowKundaliEntry(false)} />
)}
```

- [ ] **Step 4: Copy to preview and verify transition**

Tap ✨ Kundali — starfield + "Apka Kundali Jagat" should fade in for 1.8s then fade out. Tapping 🌅 Aaj switches back instantly (no animation needed for return).

- [ ] **Step 5: Commit**
```bash
git add astro-home-daily-v2.html
git commit -m "feat: add cinematic kundali entry transition"
```

---

### Task 3: Port Kundali Data Constants

**Files:**
- Modify: `astro-home-daily-v2.html` — add data constants section

Port these data blocks verbatim from `astro-home.html`:

- [ ] **Step 1: Copy `ORBIT_PLANETS` array** (with `storyBeats` and `chatReplies`)

From `astro-home.html` find `const ORBIT_PLANETS` and copy the full array into `astro-home-daily-v2.html` after the existing data constants section. Prefix it with a comment: `/* ── KUNDALI TAB DATA ── */`

- [ ] **Step 2: Copy `STORY_CARDS` array**

From `astro-home.html` find `const STORY_CARDS` and copy into the same section.

- [ ] **Step 3: Copy `LIFE_ERAS`, `DESTINY_GATES`, and helper functions**

Copy `LIFE_ERAS`, `DESTINY_GATES`, `getEraForAge`, `getGateForAge`, `getYearReading` from `astro-home.html`.

- [ ] **Step 4: Note on celebrity data** — it is embedded inside `CelebrityKundliMatch` component itself (no separate constant). No action needed here; it gets ported with the component in Task 4.

- [ ] **Step 5: Verify no Babel parse errors**
```bash
# Check preview loads without console errors
open http://localhost:8080/astro-home-daily-v2.html
```

- [ ] **Step 6: Commit**
```bash
git add astro-home-daily-v2.html
git commit -m "feat: port kundali data constants into daily page"
```

---

### Task 4: Port Kundali Components

**Files:**
- Modify: `astro-home-daily-v2.html` — add ported components

Port these components verbatim from `astro-home.html`, placed in a clearly marked section `/* ══ KUNDALI TAB COMPONENTS ══ */`:

- [ ] **Step 1: Port `useNarration` hook + `AvatarFigure` + `LivingSolarSystem`**

These three are tightly coupled. Copy from `astro-home.html`. `LivingSolarSystem` is the Graha Mandal orbit component.

- [ ] **Step 2: Port `PlanetStoryCapsule`** (planet tap → story beats + chat overlay)

This is the full 3-phase planet experience. Copy verbatim — it depends on `ORBIT_PLANETS`, `useNarration`, `matchPlanetChatCategory`.

- [ ] **Step 3: Port `matchPlanetChatCategory` helper**

- [ ] **Step 4: Port `StoryCardPreview` + `StoryOverlay`**

Story cards with the full overlay experience.

- [ ] **Step 5: Port `DestinyWalk` + `YearCapsuleOverlay` + `MonthlyCalendarView`**

The life graph + year drill-down + monthly calendar.

- [ ] **Step 6: Port `SectionHeader`** — this component does NOT exist in astro-home-daily-v2.html but is required by KundaliTab. Copy from astro-home.html.

- [ ] **Step 7: Port `CelebrityKundliMatch`**

The Kundli Twin card (already uses ESPNcricinfo Virat Kohli image).

- [ ] **Step 7: Port `AskTheStars`**

The freeform question interface.

- [ ] **Step 8: Verify no Babel parse errors after porting**

Copy to preview and confirm page loads cleanly:
```bash
cp /Users/shivali.wason/Documents/devotion/astro-home-daily-v2.html /tmp/devotion_preview/astro-home-daily-v2.html
```

- [ ] **Step 9: Commit**
```bash
git add astro-home-daily-v2.html
git commit -m "feat: port kundali components (planets, stories, life graph, twin, ask)"
```

---

### Task 5: Build KundaliTab + Wire Into App

**Files:**
- Modify: `astro-home-daily-v2.html` — KundaliTab component + App render logic

- [ ] **Step 1: Add `KundaliTab` component**

```jsx
const KundaliTab = ({ kundaliOverlay, setKundaliOverlay, initialSection }) => {
  const scrollRef = useRef(null);

  // Scroll to section if deep-linked
  useEffect(() => {
    if (initialSection && scrollRef.current) {
      const el = scrollRef.current.querySelector('[data-section="' + initialSection + '"]');
      if (el) el.scrollIntoView({ behavior: 'smooth', block: 'start' });
    }
  }, [initialSection]);

  return (
    <div ref={scrollRef} style={{
      height: '100%', overflowY: 'auto', paddingBottom: 80,
      background: '#0a0a1a',
    }}>
      {/* Hero identity strip */}
      <div style={{
        padding: '20px 20px 12px',
        background: 'linear-gradient(180deg, #0d0d2e 0%, #0a0a1a 100%)',
      }}>
        <div style={{ fontSize: 11, fontWeight: 700, color: 'rgba(255,255,255,0.3)', letterSpacing: 2, textTransform: 'uppercase', marginBottom: 4 }}>Aapki Cosmic Identity</div>
        <div style={{ fontSize: 22, fontWeight: 800, color: '#fff' }}>Shivali ♏</div>
        <div style={{ fontSize: 12, color: 'rgba(255,255,255,0.4)', marginTop: 2 }}>Vrishchik Rashi · Anuradha Nakshatra · Mulank 9</div>
      </div>

      {/* Graha Mandal */}
      <div data-section="planets">
        <SectionHeader title="Apka Graha Mandal" subtitle="Tap karke suniye — aapke grah kya keh rahe hain" delay={0} />
        <LivingSolarSystem onPlanetTap={(planet) =>
          setKundaliOverlay({ type: 'planet', planet, index: ORBIT_PLANETS.indexOf(planet) })
        } />
      </div>

      {/* Story Cards */}
      <div data-section="stories">
        <SectionHeader title="Kundli ke Rahasya" subtitle="Tap karke jaano apne baare mein" delay={100} />
        <div className="horiz-scroll" style={{ display: 'flex', gap: 12, padding: '4px 16px 16px', overflowX: 'auto' }}>
          {STORY_CARDS.map((card, i) => (
            <StoryCardPreview key={card.id} card={card} index={i}
              onTap={(idx) => setKundaliOverlay({ type: 'story', index: idx })} />
          ))}
        </div>
      </div>

      {/* Kundli Twin */}
      <div data-section="twin">
        <CelebrityKundliMatch />
      </div>

      {/* Life Graph */}
      <div data-section="lifegraph">
        <SectionHeader title="Meri Life Graph" subtitle="Past se future tak — apni life ka cosmic map" delay={200} />
        <DestinyWalk onYearTap={(age) => setKundaliOverlay({ type: 'yearCapsule', age })} />
      </div>

      {/* Ask the Stars */}
      <div data-section="ask">
        <div style={{ borderTop: '1px solid var(--stroke-minimal)', margin: '0 16px 20px' }} />
        <AskTheStars />
      </div>
    </div>
  );
};
```

- [ ] **Step 2: Update App render to show Aaj vs Kundali tab content**

Replace the current single scroll-feed with a conditional render:

```jsx
{/* Aaj Tab */}
<div style={{
  position: 'absolute', inset: 0,
  opacity: activeTab === 'aaj' ? 1 : 0,
  pointerEvents: activeTab === 'aaj' ? 'all' : 'none',
  transition: 'opacity 300ms ease',
}}>
  {/* existing scroll-feed content */}
</div>

{/* Kundali Tab */}
<div style={{
  position: 'absolute', inset: 0,
  opacity: activeTab === 'kundali' ? 1 : 0,
  pointerEvents: activeTab === 'kundali' ? 'all' : 'none',
  transition: 'opacity 300ms ease',
}}>
  <KundaliTab
    kundaliOverlay={kundaliOverlay}
    setKundaliOverlay={setKundaliOverlay}
    initialSection={kundaliSection}
  />
</div>
```

- [ ] **Step 3: Render kundali overlays (planet, story, yearCapsule)**

After the tab content divs, add overlay rendering driven by `kundaliOverlay` state (same pattern as astro-home.html's overlay rendering):

```jsx
{kundaliOverlay?.type === 'planet' && (
  <PlanetStoryCapsule
    planet={kundaliOverlay.planet}
    currentIndex={kundaliOverlay.index}
    total={ORBIT_PLANETS.length}
    onClose={() => setKundaliOverlay(null)}
    onPrev={kundaliOverlay.index > 0 ? () => setKundaliOverlay({ type: 'planet', planet: ORBIT_PLANETS[kundaliOverlay.index - 1], index: kundaliOverlay.index - 1 }) : null}
    onNext={kundaliOverlay.index < ORBIT_PLANETS.length - 1 ? () => setKundaliOverlay({ type: 'planet', planet: ORBIT_PLANETS[kundaliOverlay.index + 1], index: kundaliOverlay.index + 1 }) : null}
  />
)}
{kundaliOverlay?.type === 'story' && (
  <StoryOverlay startIndex={kundaliOverlay.index} onClose={() => setKundaliOverlay(null)} />
)}
{kundaliOverlay?.type === 'yearCapsule' && (
  <YearCapsuleOverlay age={kundaliOverlay.age} onClose={() => setKundaliOverlay(null)} />
)}
```

- [ ] **Step 4: Copy to preview and test full tab switching**
```bash
cp /Users/shivali.wason/Documents/devotion/astro-home-daily-v2.html /tmp/devotion_preview/astro-home-daily-v2.html
```

Verify:
- 🌅 Aaj shows daily content
- ✨ Kundali shows cinematic entry then graha mandal, stories, life graph, twin, ask
- Tapping a planet in Kundali tab opens PlanetStoryCapsule
- Tapping a story card opens StoryOverlay
- Tapping a year opens YearCapsuleOverlay

- [ ] **Step 5: Commit**
```bash
git add astro-home-daily-v2.html
git commit -m "feat: build KundaliTab component and wire into App with overlay handling"
```

---

### Task 6: Contextual Deep-Links from Aaj → Kundali

**Files:**
- Modify: `astro-home-daily-v2.html` — OrbitClock planet tap + energy card taps

- [ ] **Step 1: Add `onPlanetDeepLink` prop to OrbitClock**

In the OrbitClock component, when a planet node is tapped in the clock ring (not the main center planet), add an option to go to Kundali:

After the existing planet tap that plays audio, add a long-press or secondary tap that deep-links. For simplicity in the prototype, add a small "→ Explore" chip that appears when a planet slot is active:

```jsx
<div onClick={() => {
  // find matching ORBIT_PLANET and deep-link
  const match = ORBIT_PLANETS.find(p => p.nameHi === slot.nameHi || p.planet.toLowerCase() === slot.planet.toLowerCase());
  if (match && onPlanetDeepLink) onPlanetDeepLink(match);
}} style={{
  fontSize: 9, fontWeight: 700, color: '#4ecdc4',
  padding: '3px 8px', borderRadius: 8,
  background: 'rgba(78,205,196,0.1)', border: '1px solid rgba(78,205,196,0.2)',
  cursor: 'pointer', marginTop: 6,
}}>
  ✨ Explore in Kundali →
</div>
```

- [ ] **Step 2: Wire deep-link in App**

Pass handler to OrbitClock:
```jsx
<OrbitClock onPlanetDeepLink={(planet) => {
  setKundaliSection('planets');
  setKundaliOverlay({ type: 'planet', planet, index: ORBIT_PLANETS.indexOf(planet) });
  setActiveTab('kundali');
  setShowKundaliEntry(true);
}} />
```

- [ ] **Step 3: Copy to preview and test deep-link**

Tap a planet in the orbit clock → "✨ Explore in Kundali →" chip appears → tap it → cinematic entry plays → Kundali tab opens directly to that planet's story.

- [ ] **Step 4: Commit**
```bash
git add astro-home-daily-v2.html
git commit -m "feat: add contextual deep-links from orbit clock to kundali tab"
```

---

### Task 7: Final Polish + Deploy

- [ ] **Step 1: Remove `KundliExploreCard`** from Aaj tab (it was the old widget — now redundant since Kundali tab exists)

Find `KundliExploreCard` conditional render and remove it.

- [ ] **Step 2: Add `SectionHeader` component** if not already present in daily page (copy from astro-home.html)

- [ ] **Step 3: Final copy to preview + full smoke test**
```bash
cp /Users/shivali.wason/Documents/devotion/astro-home-daily-v2.html /tmp/devotion_preview/astro-home-daily-v2.html
```

Check:
- [ ] Bottom nav visible on both tabs
- [ ] Aaj tab: daily content, no overflow/clipping
- [ ] Kundali tab: cinematic entry on first switch
- [ ] Kundali tab: all 5 sections visible (planets, stories, twin, life graph, ask)
- [ ] Planet tap in Kundali opens story capsule
- [ ] Year tap opens year capsule
- [ ] Deep-link from orbit clock works
- [ ] No console errors

- [ ] **Step 4: Push to git and deploy**
```bash
git add astro-home-daily-v2.html
git commit -m "feat: returning user two-tab experience — Aaj + Kundali with cinematic transition"
git push
```

Live at: `https://shivali95.github.io/devotion/astro-home-daily-v2.html`
