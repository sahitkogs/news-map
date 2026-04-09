# Sidebar Bottom Gap — Unresolved

## Problem

The YouTube tab in the sidebar leaves visible empty space at the bottom, below the last video grid. The Articles tab does not have this issue.

## Why Articles Works

Articles are block-level `<a>` elements that stack vertically with `border-bottom` separators. They flow continuously and naturally fill the width. The content is a flat list — no wrapper with its own layout constraints.

## Why YouTube Breaks

The YouTube tab uses `.video-grid` with `display: flex; flex-wrap: wrap` and fixed `220px` wide cards. After the last row of cards, the flex container ends at its content height. The remaining sidebar-body height below is empty.

## What Was Tried

1. **Matching backgrounds** — Set `bg-surface` on sidebar, sidebar-body, video-grid, and articles. Removed redundant layered backgrounds. The empty area still appears visually different or noticeably empty.

2. **Video-feed flex wrapper** — Wrapped all video output in a `.video-feed` div with `display: flex; flex-direction: column; min-height: 100%`. Gave the last `.video-grid` element `flex: 1` to stretch. Didn't resolve the gap.

3. **More content** — Increased videos per location from 6 to 10, articles from 5 to 8. Helps fill more space but doesn't eliminate the gap when content is shorter than the sidebar height.

4. **Sidebar layout changes** — Tried `align-items: stretch` on `.main-layout`, `flex: 1 1 0` with `min-height: 0` on `.sidebar-body`, `overflow-y: auto` for scrolling.

## Current CSS Structure

```
.sidebar (flex column, overflow: hidden, bg-surface)
  .sidebar-drag-zone (flex-shrink: 0, bg-surface)
    .sidebar-drag-handle
    .sidebar-header (tabs: Articles | YouTube)
  .sidebar-body (flex: 1 1 0, overflow-y: auto, bg-surface, min-height: 0)
    [content rendered here]
```

For YouTube tab, the content is:
```
.video-feed (flex column, min-height: 100%)
  .time-group-header
  .video-grid (flex-wrap, 220px cards)
  .time-group-header
  .video-grid.video-grid-last (flex: 1)
```

## Possible Root Causes to Investigate

1. **flex-wrap + flex: 1 interaction** — A flex container with `flex-wrap: wrap` and `flex: 1` grows its height but the cards inside still align to the top. The stretched area below the last row of cards is empty flex space, not missing content.

2. **min-height: 100% on video-feed** — The `100%` refers to the sidebar-body's height, which itself is determined by flex. This might not resolve correctly in all browsers.

3. **overflow-y: auto creates a new BFC** — The sidebar-body's `overflow-y: auto` creates a block formatting context. The `min-height: 100%` on `.video-feed` might not reference the scrollable area's full height.

## Ideas for Next Session

- **CSS Grid instead of flexbox** for video layout — `grid-template-rows` can fill space more predictably than flex-wrap
- **Use `align-content: start`** on the video-grid so cards pin to top, then just ensure the container background matches
- **JavaScript approach** — after render, calculate remaining height and set it as padding-bottom or min-height on the last grid
- **Accept the gap and mask it** — use a gradient fade-out at the bottom of the sidebar-body instead of trying to fill it
- **Switch video layout to vertical list** (like articles) instead of a grid — this naturally fills width and avoids the flex-wrap gap issue entirely
