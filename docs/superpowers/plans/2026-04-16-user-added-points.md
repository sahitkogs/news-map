# User-Added Map Points — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Transform the hardcoded Amaravati tracker into a personal news map where users search for any place, add it, and get news/video feeds.

**Architecture:** User locations stored in localStorage, loaded on startup, rendered as markers. Leaflet Control Geocoder provides search. Sidebar confirmation panel handles the add flow. Existing Worker pipeline unchanged — it already accepts any query string.

**Tech Stack:** Vanilla JS, Leaflet, Leaflet Control Geocoder (Nominatim), Cloudflare Workers (unchanged)

**Spec:** `docs/superpowers/specs/2026-04-16-user-added-points-design.md`

---

## File Structure

| File | Role | Change |
|---|---|---|
| `index.html` | Page shell | Add geocoder CDN, update branding, remove category filters, remove data.js |
| `styles.css` | Styles | Add geocoder/confirmation/empty-state CSS, remove dead category filter CSS |
| `app.js` | Client logic | Rewrite: user location CRUD, geocoder, confirmation panel, remove category/filter code |
| `data.js` | Static data | **Delete** |
| `tests/client-cache-test.js` | Integration test | Update to seed a user location instead of relying on hardcoded LOCATIONS |

---

### Task 1: Update index.html

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add geocoder CDN links**

Add after the Leaflet CSS link (line 9):

```html
<link rel="stylesheet" href="https://unpkg.com/leaflet-control-geocoder@2.4.0/dist/Control.Geocoder.css" />
```

Add after the Leaflet JS script (line 80):

```html
<script src="https://unpkg.com/leaflet-control-geocoder@2.4.0/dist/Control.Geocoder.js"></script>
```

- [ ] **Step 2: Update branding**

Replace the topbar-brand div (lines 15-18):

```html
  <div class="topbar-brand">
    <span class="topbar-title">News Map</span>
    <span class="topbar-subtitle">Track news &amp; videos for any place</span>
  </div>
```

- [ ] **Step 3: Remove category filters and dividers**

Remove the entire filters section and both dividers around it (lines 19-44):

```html
  <div class="topbar-divider"></div>
  <div class="filters" id="filters">
    ...all filter buttons...
  </div>
  <div class="filter-dropdown" id="filterDropdown">
    ...all dropdown items...
  </div>
  <div class="topbar-divider"></div>
```

Replace with a point counter and single divider:

```html
  <div class="topbar-spacer"></div>
  <div class="point-counter" id="pointCounter">0 / 10 points</div>
  <div class="topbar-divider"></div>
```

- [ ] **Step 4: Update page title**

Change `<title>` (line 6):

```html
<title>News Map</title>
```

- [ ] **Step 5: Remove data.js script tag**

Delete line 81:

```html
<script src="data.js"></script>
```

- [ ] **Step 6: Verify and commit**

Open http://localhost:8766/index.html — page will show errors in console (app.js still references LOCATIONS). That's expected. Verify:
- Geocoder CSS/JS loads (check Network tab)
- Branding says "News Map"
- No category filter pills visible
- Point counter "0 / 10 points" visible in header

```bash
git add index.html
git commit -m "feat: update index.html for user-added points

Remove category filters, update branding to News Map, add geocoder CDN,
remove data.js script tag, add point counter to header."
```

---

### Task 2: Add new CSS

**Files:**
- Modify: `styles.css`

- [ ] **Step 1: Add topbar-spacer and point-counter styles**

Add after the existing `.filter-dropdown-item .dot` rule (after line 253):

```css
/* ── Point Counter (header) ── */
.topbar-spacer {
  flex: 1;
}

.point-counter {
  font-size: 0.72rem;
  color: var(--text-muted);
  background: var(--bg-elevated);
  padding: 4px 12px;
  border-radius: 100px;
  border: 1px solid var(--border);
  white-space: nowrap;
  flex-shrink: 0;
}
```

- [ ] **Step 2: Add geocoder theme overrides**

Add after the `.leaflet-control-zoom a:hover` rule (after line 376):

```css
/* ── Geocoder Control ── */
.leaflet-control-geocoder {
  background: var(--bg-surface) !important;
  border: 1px solid var(--border) !important;
  border-radius: var(--radius) !important;
  box-shadow: 0 2px 8px rgba(0,0,0,0.3) !important;
}

.leaflet-control-geocoder input {
  background: var(--bg-surface) !important;
  color: var(--text-primary) !important;
  font-family: 'Plus Jakarta Sans', system-ui, sans-serif !important;
  font-size: 0.82rem !important;
}

.leaflet-control-geocoder input::placeholder {
  color: var(--text-muted) !important;
}

.leaflet-control-geocoder-alternatives {
  background: var(--bg-surface) !important;
  border: 1px solid var(--border) !important;
  border-top: none !important;
}

.leaflet-control-geocoder-alternatives li {
  color: var(--text-primary) !important;
  font-family: 'Plus Jakarta Sans', system-ui, sans-serif !important;
  font-size: 0.78rem !important;
  border-bottom: 1px solid var(--border) !important;
}

.leaflet-control-geocoder-alternatives li:hover,
.leaflet-control-geocoder-selected {
  background: var(--bg-hover) !important;
}

.leaflet-control-geocoder-icon {
  filter: invert(0.7);
}

body.day-mode .leaflet-control-geocoder {
  background: #fff !important;
  border-color: #ccc !important;
}

body.day-mode .leaflet-control-geocoder input {
  background: #fff !important;
  color: #333 !important;
}

body.day-mode .leaflet-control-geocoder-alternatives {
  background: #fff !important;
  border-color: #ccc !important;
}

body.day-mode .leaflet-control-geocoder-alternatives li {
  color: #333 !important;
  border-color: #eee !important;
}

body.day-mode .leaflet-control-geocoder-alternatives li:hover {
  background: #f0f0f0 !important;
}

body.day-mode .leaflet-control-geocoder-icon {
  filter: none;
}
```

- [ ] **Step 3: Add confirmation panel styles**

Add after the geocoder styles:

```css
/* ── Confirmation Panel ── */
.confirmation-panel {
  padding: 24px 20px;
}

.confirmation-label {
  font-size: 0.68rem;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  color: var(--text-muted);
  margin-bottom: 12px;
  font-weight: 600;
}

.confirmation-name {
  font-family: 'DM Serif Display', serif;
  font-size: 1.1rem;
  color: var(--text-primary);
  margin-bottom: 4px;
}

.confirmation-coords {
  font-size: 0.72rem;
  color: var(--text-muted);
  margin-bottom: 16px;
}

.confirmation-field-label {
  display: block;
  font-size: 0.72rem;
  color: var(--text-secondary);
  margin-bottom: 6px;
  font-weight: 600;
}

.confirmation-input {
  width: 100%;
  padding: 8px 12px;
  background: var(--bg-elevated);
  border: 1px solid var(--border-light);
  border-radius: var(--radius);
  color: var(--text-primary);
  font-family: inherit;
  font-size: 0.82rem;
  outline: none;
  transition: border-color var(--transition);
}

.confirmation-input:focus {
  border-color: var(--accent-gold);
}

.confirmation-counter {
  font-size: 0.72rem;
  color: var(--text-muted);
  margin-top: 12px;
}

.confirmation-limit {
  font-size: 0.75rem;
  color: var(--cat-government);
  margin-top: 8px;
  line-height: 1.4;
}

.confirmation-actions {
  display: flex;
  gap: 8px;
  margin-top: 16px;
}

.confirmation-btn {
  padding: 8px 20px;
  border-radius: var(--radius);
  border: none;
  font-family: inherit;
  font-size: 0.78rem;
  font-weight: 600;
  cursor: pointer;
  transition: all var(--transition);
}

.confirmation-btn.add {
  background: var(--accent-gold);
  color: #000;
}

.confirmation-btn.add:hover:not(:disabled) {
  background: var(--accent-gold-dim);
}

.confirmation-btn.add:disabled {
  opacity: 0.4;
  cursor: not-allowed;
}

.confirmation-btn.cancel {
  background: var(--bg-elevated);
  color: var(--text-secondary);
  border: 1px solid var(--border);
}

.confirmation-btn.cancel:hover {
  background: var(--bg-hover);
  color: var(--text-primary);
}
```

- [ ] **Step 4: Add empty state styles**

```css
/* ── Empty State ── */
.empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 48px 32px;
  text-align: center;
  min-height: 300px;
}

.empty-state-icon {
  font-size: 2.5rem;
  margin-bottom: 16px;
  opacity: 0.5;
}

.empty-state-title {
  font-family: 'DM Serif Display', serif;
  font-size: 1.05rem;
  color: var(--text-primary);
  margin-bottom: 8px;
}

.empty-state-text {
  font-size: 0.78rem;
  color: var(--text-muted);
  line-height: 1.5;
  max-width: 240px;
}

.empty-state-counter {
  margin-top: 16px;
  font-size: 0.72rem;
  color: var(--text-muted);
}
```

- [ ] **Step 5: Add remove button styles for marker popup**

```css
/* ─��� Remove Button in Popup ── */
.remove-point-btn {
  display: block;
  width: 100%;
  margin-top: 10px;
  padding: 5px 0;
  background: transparent;
  border: 1px solid var(--cat-government);
  border-radius: 4px;
  color: var(--cat-government);
  font-family: inherit;
  font-size: 0.7rem;
  font-weight: 600;
  cursor: pointer;
  transition: all var(--transition);
}

.remove-point-btn:hover {
  background: var(--cat-government);
  color: #fff;
}
```

- [ ] **Step 6: Add temp marker pulse animation**

```css
/* ── Temp Marker (pending add) ── */
.temp-marker {
  animation: pulse 1.5s ease-in-out infinite;
}

@keyframes pulse {
  0%, 100% { box-shadow: 0 2px 8px rgba(0,0,0,0.4), 0 0 0 1px rgba(0,0,0,0.1); }
  50% { box-shadow: 0 2px 8px rgba(74,158,218,0.6), 0 0 12px rgba(74,158,218,0.3); }
}
```

- [ ] **Step 7: Commit**

```bash
git add styles.css
git commit -m "feat: add CSS for geocoder, confirmation panel, empty state, remove button"
```

---

### Task 3: Rewrite app.js — user location CRUD and markers

**Files:**
- Modify: `app.js`

This task replaces the LOCATIONS-based marker system with user-location-based markers and adds the CRUD functions. The geocoder and confirmation panel are added in Task 4.

- [ ] **Step 1: Update default map center to world view**

Replace lines 4-8:

```js
const map = L.map('map', {
  center: [20, 0],
  zoom: 2,
  zoomControl: true,
  attributionControl: true
});
```

- [ ] **Step 2: Replace MARKERS section with user locations**

Replace the entire `MARKERS & LAYER GROUPS` section (lines 62-129) with:

```js
// ══════════════════════════════════════════════════════════
//  USER LOCATIONS — localStorage persistence
// ═════════��════════════════════════════════════════════════
const MAX_POINTS = 10;
const MARKER_COLOR = '#d4a857';

function loadUserLocations() {
  try {
    const raw = localStorage.getItem('userLocations');
    return raw ? JSON.parse(raw) : [];
  } catch { return []; }
}

function saveUserLocations(locs) {
  localStorage.setItem('userLocations', JSON.stringify(locs));
}

let userLocations = loadUserLocations();

// ══════════════════════════════════════════════════════════
//  MARKERS
// ═════════════════════════════════���════════════════════════
const markerGroup = L.layerGroup().addTo(map);
const markersById = new Map();

function createMarker(loc) {
  const size = 12;
  const icon = L.divIcon({
    className: '',
    html: `<div class="marker-icon" style="width:${size}px;height:${size}px;background:${MARKER_COLOR};"></div>`,
    iconSize: [size, size],
    iconAnchor: [size / 2, size / 2]
  });
  const marker = L.marker([loc.lat, loc.lng], { icon }).addTo(markerGroup);

  const popupContent = `
    <div class="map-popup">
      <div class="map-popup-name">${loc.name}</div>
      <div class="map-popup-desc">${loc.searchKeywords}</div>
      <button class="remove-point-btn" data-loc-id="${loc.id}">Remove</button>
    </div>
  `;

  marker.bindTooltip(loc.name, {
    direction: 'top',
    offset: [0, -8],
    className: 'marker-tooltip',
    opacity: 1
  });

  marker.bindPopup(popupContent, {
    className: 'custom-popup',
    maxWidth: 260,
    closeButton: false
  });

  marker.on('click', () => {
    cancelPending();
    selectedLocation = loc;
    lastVisibleIdsByTab.articles = '';
    lastVisibleIdsByTab.videos = '';
    renderSidebar(true);
    map.setView([loc.lat, loc.lng], Math.max(map.getZoom(), 14), { animate: true });
  });

  markersById.set(loc.id, marker);
}

function removeLocation(id) {
  const marker = markersById.get(id);
  if (marker) {
    map.closePopup();
    markerGroup.removeLayer(marker);
    markersById.delete(id);
  }
  userLocations = userLocations.filter(l => l.id !== id);
  saveUserLocations(userLocations);
  if (selectedLocation && selectedLocation.id === id) {
    selectedLocation = null;
  }
  articlesByLoc.delete(id);
  videosByLoc.delete(id);
  updatePointCounter();
  renderSidebar(true);
}

function addLocation(loc) {
  userLocations.push(loc);
  saveUserLocations(userLocations);
  createMarker(loc);
  updatePointCounter();
}

function updatePointCounter() {
  const el = document.getElementById('pointCounter');
  if (el) el.textContent = `${userLocations.length} / ${MAX_POINTS} points`;
}

// Handle remove button clicks inside popups (event delegation)
map.on('popupopen', (e) => {
  const container = e.popup.getElement();
  if (!container) return;
  const btn = container.querySelector('.remove-point-btn');
  if (btn) {
    btn.addEventListener('click', () => removeLocation(btn.dataset.locId));
  }
});

// Clear selection when popup closes
map.on('popupclose', () => {
  if (selectedLocation) {
    selectedLocation = null;
    lastVisibleIdsByTab.articles = '';
    lastVisibleIdsByTab.videos = '';
    renderSidebar(true);
  }
});

// Initialize existing markers
userLocations.forEach(createMarker);
updatePointCounter();
```

- [ ] **Step 3: Add pending-location helpers (stub for Task 4)**

Add immediately after the marker initialization code, before the NEWS FETCHING section:

```js
// ════════════════════════════════��════════════════════��════
//  GEOCODER — placeholder, wired up in next task
// ═══════════════════════════════���══════════════════════════
let pendingLocation = null;
let tempMarker = null;

function cancelPending() {
  if (tempMarker) { map.removeLayer(tempMarker); tempMarker = null; }
  pendingLocation = null;
}
```

- [ ] **Step 4: Update getVisibleLocations — remove filter reference**

Replace `getVisibleLocations` (lines 262-271 of original):

```js
function getVisibleLocations() {
  if (selectedLocation) return [selectedLocation];
  const bounds = map.getBounds();
  return userLocations.filter(loc => bounds.contains([loc.lat, loc.lng]));
}
```

- [ ] **Step 5: Update renderArticleHtml — remove category color**

Replace `renderArticleHtml` function. Change the tag color from `CATEGORY_COLORS[article._loc.category]` to `MARKER_COLOR`:

```js
function renderArticleHtml(article) {
  const tagColor = MARKER_COLOR;
  const thumbHtml = article.thumb
    ? `<img class="news-article-thumb" src="${article.thumb}" alt="" loading="lazy" onerror="this.remove()">`
    : '';

  return `
    <a class="news-article" href="${article.link}" target="_blank" rel="noopener">
      <div class="news-article-body">
        <div class="news-article-source">
          <span class="news-article-source-name">${article.source}</span>
          <span class="news-article-time">${timeAgo(article.pubDate)}</span>
        </div>
        <div class="news-article-title">${article.title}</div>
        <span class="news-article-tag" style="background:${tagColor}18;color:${tagColor};">
          <span class="news-article-tag-dot" style="background:${tagColor};"></span>
          ${article._loc.name}
        </span>
      </div>
      ${thumbHtml}
    </a>
  `;
}
```

- [ ] **Step 6: Update renderVideoHtml — remove category color**

Replace `renderVideoHtml` function. Same change — use `MARKER_COLOR` instead of `CATEGORY_COLORS`:

```js
function renderVideoHtml(video) {
  const tagColor = MARKER_COLOR;
  const dur = formatDuration(video.duration);
  const views = formatViews(video.views);

  return `
    <a class="video-card" href="${video.link}" target="_blank" rel="noopener">
      <div class="video-card-thumb">
        <img src="${video.thumb}" alt="" loading="lazy" onerror="this.parentElement.style.display='none'">
        ${dur ? `<span class="video-card-duration">${dur}</span>` : ''}
      </div>
      <div class="video-card-title">${video.title}</div>
      <div class="video-card-meta">
        <span class="video-card-channel">${video.channel}</span>
        ${views ? `<span class="sep"></span><span>${views}</span>` : ''}
        ${video.published ? `<span class="sep"></span><span>${timeAgo(video.published)}</span>` : ''}
      </div>
      <span class="news-article-tag" style="background:${tagColor}18;color:${tagColor};">
        <span class="news-article-tag-dot" style="background:${tagColor};"></span>
        ${video._loc.name}
      </span>
    </a>
  `;
}
```

- [ ] **Step 7: Update renderSidebar — add empty state**

In `renderSidebar`, replace the `visible.length === 0` block (line 411-416 of original):

```js
  if (userLocations.length === 0) {
    lastVisibleIdsByTab.articles = '';
    lastVisibleIdsByTab.videos = '';
    sidebarBody.innerHTML = `
      <div class="empty-state">
        <div class="empty-state-icon">&#128204;</div>
        <div class="empty-state-title">No points yet</div>
        <div class="empty-state-text">Search for a place and add it to your map to start tracking news and videos.</div>
        <div class="empty-state-counter">${userLocations.length} / ${MAX_POINTS} points</div>
      </div>
    `;
    return;
  }

  if (visible.length === 0) {
    lastVisibleIdsByTab.articles = '';
    lastVisibleIdsByTab.videos = '';
    sidebarBody.innerHTML = `<div class="sidebar-empty">No points in the current view.<br>Zoom out or pan the map.</div>`;
    return;
  }
```

- [ ] **Step 8: Update HOME BUTTON defaults**

Replace `DEFAULT_CENTER` and `DEFAULT_ZOOM`:

```js
const DEFAULT_CENTER = [20, 0];
const DEFAULT_ZOOM = 2;
```

- [ ] **Step 9: Delete category filter code**

Remove the entire `CATEGORY FILTERS (desktop pills)` section (original lines 553-587) — the `applyFilter` function and the `#filters` click handler.

Remove the entire `FILTER DROPDOWN (mobile)` section (original lines 589-611) — `filterToggle`, `filterMenu`, and all three event listeners.

- [ ] **Step 10: Verify in browser**

Open http://localhost:8766/index.html. Verify:
- Map starts at world view (zoom 2)
- No JS errors in console (except geocoder not initialized yet — that's Task 4)
- Empty state shows in sidebar: pin icon, "No points yet", "0 / 10 points"
- Header shows "News Map" and "0 / 10 points" counter
- No category filter UI anywhere
- If you manually add a location to localStorage via console:
  ```js
  localStorage.setItem('userLocations', JSON.stringify([{id:'test_1',name:'Test',lat:35.6,lng:139.7,searchKeywords:'Tokyo news',addedAt:Date.now()}]));
  location.reload();
  ```
  - Marker appears on map
  - Clicking it shows popup with name, keywords, and Remove button
  - Remove button works (marker disappears, localStorage updated)

- [ ] **Step 11: Commit**

```bash
git add app.js
git commit -m "feat: replace LOCATIONS with user location CRUD

User locations stored in localStorage, markers created dynamically.
Remove category filters and hardcoded LOCATIONS references. Add empty
state, remove-point popup, and point counter."
```

---

### Task 4: Add geocoder and confirmation panel

**Files:**
- Modify: `app.js`

- [ ] **Step 1: Wire up geocoder control**

Replace the geocoder placeholder section (added in Task 3 step 3) with the full implementation:

```js
// ══════════════════════��══════════════════════════════���════
//  GEOCODER — search bar + confirmation panel
// ═══���══════════════════════════════════════════════════════
let pendingLocation = null;
let tempMarker = null;

function cancelPending() {
  if (tempMarker) { map.removeLayer(tempMarker); tempMarker = null; }
  pendingLocation = null;
}

const geocoderControl = L.Control.geocoder({
  geocoder: L.Control.Geocoder.nominatim(),
  placeholder: 'Search for a place...',
  defaultMarkGeocode: false,
  position: 'topleft',
  collapsed: false
}).addTo(map);

geocoderControl.on('markgeocode', (e) => {
  const { center, name } = e.geocode;

  // Clean up any previous pending state
  cancelPending();
  if (selectedLocation) {
    selectedLocation = null;
    map.closePopup();
  }

  // Place temporary marker
  tempMarker = L.marker(center, {
    icon: L.divIcon({
      className: '',
      html: '<div class="marker-icon temp-marker" style="width:14px;height:14px;background:#4a9eda;"></div>',
      iconSize: [14, 14],
      iconAnchor: [7, 7]
    })
  }).addTo(map);

  map.setView(center, Math.max(map.getZoom(), 12), { animate: true });

  pendingLocation = { name, lat: center.lat, lng: center.lng };
  showConfirmationPanel(pendingLocation);
});
```

- [ ] **Step 2: Add showConfirmationPanel function**

Add after the geocoder code:

```js
function showConfirmationPanel(location) {
  const atLimit = userLocations.length >= MAX_POINTS;
  sidebarBody.innerHTML = `
    <div class="confirmation-panel">
      <div class="confirmation-label">Add new point</div>
      <div class="confirmation-name">${location.name}</div>
      <div class="confirmation-coords">${location.lat.toFixed(4)}, ${location.lng.toFixed(4)}</div>
      <label class="confirmation-field-label">Search keywords for news &amp; videos:</label>
      <input type="text" class="confirmation-input" id="keywordsInput" value="${location.name}">
      <div class="confirmation-counter">${userLocations.length} / ${MAX_POINTS} points</div>
      ${atLimit ? '<div class="confirmation-limit">You\'ve reached the 10-point limit. Remove a point to add a new one.</div>' : ''}
      <div class="confirmation-actions">
        <button class="confirmation-btn add" id="confirmAdd" ${atLimit ? 'disabled' : ''}>Add Point</button>
        <button class="confirmation-btn cancel" id="confirmCancel">Cancel</button>
      </div>
    </div>
  `;

  document.getElementById('confirmAdd').addEventListener('click', () => {
    const keywords = document.getElementById('keywordsInput').value.trim();
    if (!keywords || userLocations.length >= MAX_POINTS) return;

    const loc = {
      id: 'user_' + Date.now(),
      name: location.name,
      lat: location.lat,
      lng: location.lng,
      searchKeywords: keywords,
      addedAt: Date.now()
    };

    cancelPending();
    addLocation(loc);
    selectedLocation = loc;
    lastVisibleIdsByTab.articles = '';
    lastVisibleIdsByTab.videos = '';
    renderSidebar(true);
    map.setView([loc.lat, loc.lng], Math.max(map.getZoom(), 14), { animate: true });
  });

  document.getElementById('confirmCancel').addEventListener('click', () => {
    cancelPending();
    renderSidebar(true);
  });
}
```

- [ ] **Step 3: Verify in browser**

Open http://localhost:8766/index.html. Test the full flow:
1. Search bar visible in top-left of map
2. Type "Tokyo Tower" — autocomplete results appear
3. Select a result — map pans, blue pulsing temp marker appears
4. Sidebar shows confirmation panel: name, coords, editable keywords, Add/Cancel buttons, "0 / 10 points"
5. Edit keywords to "Tokyo Tower tourism news"
6. Click "Add Point" — permanent gold marker appears, sidebar switches to news feed, starts loading articles
7. Click the marker — popup shows name, keywords, Remove button
8. Search another place, click Cancel — temp marker disappears, sidebar returns to feed
9. Add 10 points — on the 11th search, confirmation panel shows limit message, Add button disabled

- [ ] **Step 4: Commit**

```bash
git add app.js
git commit -m "feat: add geocoder search and confirmation panel

Leaflet Control Geocoder with Nominatim for global place search.
Selecting a result shows a sidebar confirmation panel with editable
search keywords. Enforces 10-point limit with disabled Add button."
```

---

### Task 5: Delete data.js and clean up dead CSS

**Files:**
- Delete: `data.js`
- Modify: `styles.css`

- [ ] **Step 1: Delete data.js**

```bash
rm data.js
```

- [ ] **Step 2: Remove dead category filter CSS from styles.css**

Remove these CSS blocks entirely:
- `.filters` through `.filter-btn .dot` (lines 127-175)
- `.filter-dropdown` through `.filter-dropdown-item .dot` (lines 177-253)

Also remove the mobile rule that shows/hides filters (inside `@media (max-width: 768px)`):

```css
  /* Hide pill filters, show dropdown */
  .filters { display: none; }
  .filter-dropdown { display: block; }
```

Remove the category CSS custom properties from `:root` (lines 16-21):

```css
  --cat-government: #e05555;
  --cat-road: #4a9eda;
  --cat-bridge: #e0943a;
  --cat-residential: #5bb868;
  --cat-commercial: #a07ad4;
  --cat-utility: #7a8899;
  --status-completed: #5bb868;
  --status-construction: #e0943a;
  --status-planned: #4a9eda;
  --status-stalled: #e05555;
```

Note: keep `--cat-government: #e05555;` since `styles.css` uses it in `.confirmation-limit` and `.remove-point-btn`. Actually, those reference it by name — replace those two references with a hardcoded `#e05555` or define a `--danger` variable. Simplest: replace `var(--cat-government)` with `#e05555` in the new CSS added in Task 2, then remove all category variables.

- [ ] **Step 3: Verify in browser**

No JS errors. App works exactly as before. Network tab shows no request for `data.js`.

- [ ] **Step 4: Commit**

```bash
git rm data.js
git add styles.css
git commit -m "chore: delete data.js and remove dead category filter CSS"
```

---

### Task 6: Update tests and CLAUDE.md

**Files:**
- Modify: `tests/client-cache-test.js`
- Modify: `CLAUDE.md`

- [ ] **Step 1: Rewrite client-cache-test.js**

The test needs to seed a user location into localStorage before loading the page, since there are no hardcoded locations anymore.

Replace the entire file:

```js
/**
 * Client-side cache tests
 * Verifies that localStorage caching prevents redundant Worker calls.
 *
 * Run: node tests/client-cache-test.js [url]
 * Requires: playwright (npx playwright install chromium)
 */

const { chromium } = require('playwright');

const BASE_URL = process.argv[2] || 'http://localhost:8766/index.html';
let pass = 0, fail = 0;

function assert(condition, msg) {
  if (condition) { console.log(`  PASS: ${msg}`); pass++; }
  else { console.log(`  FAIL: ${msg}`); fail++; }
}

(async () => {
  const browser = await chromium.launch();
  const context = await browser.newContext();
  const page = await context.newPage();

  console.log(`=== Client Cache Tests ===`);
  console.log(`URL: ${BASE_URL}\n`);

  // Seed a user location so the app has something to fetch
  const testLocation = {
    id: 'test_cache_1',
    name: 'Hyderabad',
    lat: 17.385,
    lng: 78.4867,
    searchKeywords: 'Hyderabad India news',
    addedAt: Date.now()
  };

  // Navigate first to set localStorage on the correct origin
  await page.goto(BASE_URL);
  await page.evaluate((loc) => {
    localStorage.setItem('userLocations', JSON.stringify([loc]));
  }, testLocation);

  // ── Test 1: localStorage gets populated after load ──
  console.log('--- Test 1: localStorage populated on first load ---');

  // Reload so the app reads the seeded location
  await page.goto(BASE_URL);

  // Pan map to the test location so it's in view
  await page.evaluate(([lat, lng]) => {
    map.setView([lat, lng], 10);
  }, [testLocation.lat, testLocation.lng]);

  // Wait for articles to finish loading
  await page.waitForTimeout(15000);

  // Click YouTube tab and wait for videos
  await page.click('.sidebar-tab:nth-child(2)');

  let videosLoaded = false;
  try {
    await page.waitForSelector('.video-card', { timeout: 30000 });
    await page.waitForTimeout(10000);
    videosLoaded = true;
  } catch {
    console.log('  INFO: No video cards appeared within 30s (YouTube API quota may be exhausted)');
  }

  const videoCache = await page.evaluate(() => {
    const raw = localStorage.getItem('videoCache');
    if (!raw) return null;
    const entries = JSON.parse(raw);
    return { entryCount: entries.length, firstKey: entries[0]?.[0] || null };
  });

  if (videosLoaded) {
    assert(videoCache !== null, 'videoCache exists in localStorage');
    assert(videoCache && videoCache.entryCount > 0, `videoCache has ${videoCache?.entryCount || 0} entries`);
  } else {
    console.log('  SKIP: videoCache test skipped (no videos loaded — likely quota exceeded)');
  }

  const newsCache = await page.evaluate(() => {
    const raw = localStorage.getItem('newsCache');
    if (!raw) return null;
    const entries = JSON.parse(raw);
    return { entryCount: entries.length, firstKey: entries[0]?.[0] || null };
  });

  assert(newsCache !== null, 'newsCache exists in localStorage');
  assert(newsCache && newsCache.entryCount > 0, `newsCache has ${newsCache?.entryCount || 0} entries`);

  console.log('');

  // ── Test 2: Second load uses cache — no Worker requests ──
  console.log('--- Test 2: Cached load makes no Worker requests ---');

  const workerRequests = [];
  await page.route('**/cors-proxy-staging.sahit-koganti.workers.dev/**', (route) => {
    workerRequests.push(route.request().url());
    route.continue();
  });
  await page.route('**/cors-proxy.sahit-koganti.workers.dev/**', (route) => {
    workerRequests.push(route.request().url());
    route.continue();
  });

  // Reload — localStorage cache should serve everything
  await page.goto(BASE_URL);
  await page.evaluate(([lat, lng]) => {
    map.setView([lat, lng], 10);
  }, [testLocation.lat, testLocation.lng]);
  await page.waitForTimeout(8000);

  // Click YouTube tab
  await page.click('.sidebar-tab:nth-child(2)');
  await page.waitForTimeout(8000);

  const videoCards = await page.$$eval('.video-card', els => els.length);

  const ytRequests = workerRequests.filter(u => u.includes('/youtube/search'));
  const newsRequests = workerRequests.filter(u => u.includes('/news/search'));

  assert(newsRequests.length === 0, `News Worker requests on reload: ${newsRequests.length} (expected 0)`);

  if (videosLoaded) {
    assert(videoCards > 0, `${videoCards} video cards rendered on cached load`);
    assert(ytRequests.length === 0, `YouTube Worker requests on reload: ${ytRequests.length} (expected 0)`);
  } else {
    console.log(`  SKIP: YouTube cache test skipped (quota exhausted, ${ytRequests.length} requests made)`);
  }

  if (workerRequests.length > 0) {
    console.log(`  INFO: Worker requests made: ${workerRequests.length}`);
    workerRequests.slice(0, 3).forEach(u => console.log(`    ${u.slice(0, 100)}`));
  }

  console.log('');

  // ── Test 3: Cache entries have valid timestamps ──
  console.log('--- Test 3: Cache entries have valid timestamps ---');

  const timestamps = await page.evaluate(() => {
    const results = {};
    ['videoCache', 'newsCache'].forEach(key => {
      const raw = localStorage.getItem(key);
      if (!raw) { results[key] = null; return; }
      const entries = JSON.parse(raw);
      results[key] = entries.map(([k, v]) => ({
        key: k.slice(0, 40),
        fetchedAt: v.fetchedAt,
        ageMinutes: Math.round((Date.now() - v.fetchedAt) / 60000),
        itemCount: (v.videos || v.articles || []).length
      }));
    });
    return results;
  });

  if (timestamps.videoCache) {
    const allValid = timestamps.videoCache.every(e => e.fetchedAt > 0 && e.ageMinutes < 130);
    assert(allValid, `All ${timestamps.videoCache.length} video cache entries have valid timestamps (< 2hr old)`);
  }
  if (timestamps.newsCache) {
    const allValid = timestamps.newsCache.every(e => e.fetchedAt > 0 && e.ageMinutes < 130);
    assert(allValid, `All ${timestamps.newsCache.length} news cache entries have valid timestamps (< 2hr old)`);
  }

  console.log('');

  // ── Cleanup ──
  await page.evaluate(() => {
    localStorage.removeItem('userLocations');
  });

  // ── Summary ──
  console.log('=== Results ===');
  console.log(`  ${pass} passed, ${fail} failed`);

  await browser.close();
  process.exit(fail > 0 ? 1 : 0);
})();
```

- [ ] **Step 2: Update CLAUDE.md**

Update the Project context section to reflect the new app:

Replace:
```
Vanilla HTML/CSS/JS static site (no build step, no framework). An interactive Leaflet map of Amaravati construction sites with a sidebar that fetches live news (Google News RSS) and YouTube videos for the locations currently in view. Long-term goal is to make this a general template — users will be able to mark their own points and see feeds for them — so avoid hardcoding Amaravati-specific assumptions outside `data.js`.
```

With:
```
Vanilla HTML/CSS/JS static site (no build step, no framework). An interactive Leaflet map where users search for any place in the world, add it as a point (up to 10), and see live news (Google News RSS) and YouTube videos for those locations. User locations are stored in localStorage. The geocoder uses Leaflet Control Geocoder (Nominatim, free, no API key).
```

Update the Files section — remove `data.js` entry, update `app.js` description:

Replace the `data.js` bullet:
```
- `data.js` — `LOCATIONS` array. The `searchKeywords` field on each location is what gets sent to both News and YouTube searches. Adding/changing locations only requires editing this file.
```

With:
```
- `data.js` — **Deleted.** User locations are now stored in localStorage and managed via the in-app geocoder search bar.
```

Update the `app.js` bullet to mention the new sections:
```
- `app.js` — Single-file client. Sections: `MAP INIT`, `USER LOCATIONS` (localStorage CRUD), `MARKERS`, `GEOCODER` (search + confirmation panel), `NEWS FETCHING`, `YOUTUBE FETCHING`, `SIDEBAR`, etc.
```

- [ ] **Step 3: Run client-cache tests**

```bash
node tests/client-cache-test.js http://localhost:8766/index.html
```

Expected: All tests pass (news cache populated, cached reload makes 0 Worker requests, valid timestamps). YouTube tests may skip if API quota is exhausted.

- [ ] **Step 4: Run edge cache tests**

```bash
bash tests/cache-tests.sh staging
```

Expected: Passes — these test the Worker, which is unchanged.

- [ ] **Step 5: Commit**

```bash
git add tests/client-cache-test.js CLAUDE.md
git commit -m "test: update client cache test for user-added points

Seed a test location into localStorage before loading the page, since
hardcoded LOCATIONS no longer exist. Also update CLAUDE.md to reflect
the new architecture."
```

---

## Verification Checklist

After all tasks are complete, verify the full flow end-to-end:

- [ ] Fresh visit (clear localStorage): map shows world view, sidebar shows empty state, header shows "News Map" and "0 / 10 points"
- [ ] Search "Tokyo Tower" — autocomplete works, selecting result shows pulsing blue temp marker and confirmation panel
- [ ] Edit keywords, click Add — permanent gold marker, sidebar loads news, counter updates to "1 / 10 points"
- [ ] Click the marker — popup with name + keywords + Remove button
- [ ] Click Remove — marker gone, sidebar updates, counter decrements
- [ ] Add 10 points — 11th search shows limit message, Add button disabled
- [ ] Refresh page — all 10 points persist (localStorage), markers re-created
- [ ] Day/night toggle and satellite toggle still work
- [ ] Mobile: sidebar drag resize works, search bar usable
- [ ] `tests/cache-tests.sh staging` passes
- [ ] `node tests/client-cache-test.js` passes (with seeded location)
