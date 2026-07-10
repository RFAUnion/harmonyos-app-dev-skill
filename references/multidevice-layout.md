# Spatialization And Multi-Device Layout

Use this reference when the user asks for HarmonyOS 7 spatialization, tablet or foldable adaptation, multi-window behavior, responsive workbench layouts, master-detail pages, or a plan for adapting an existing phone-first app.

Source case study: `https://developer.huawei.com/consumer/cn/blog/topic/03218972610058011`

## Contents

- [Core Principle](#core-principle)
- [Page Prioritization](#page-prioritization)
- [Window And Posture Strategy](#window-and-posture-strategy)
- [Home And Workbench](#home-and-workbench)
- [List And Detail](#list-and-detail)
- [Edit And Form](#edit-and-form)
- [Empty, Settings, And Low-Frequency Pages](#empty-settings-and-low-frequency-pages)
- [Spatialization And Motion Rules](#spatialization-and-motion-rules)
- [Validation Checklist](#validation-checklist)
- [Common Pitfalls](#common-pitfalls)

## Core Principle

Do not interpret spatialization as “make every page 3D” and do not interpret multi-device support as “stretch the phone layout.” For ordinary tools, office, content, meeting, learning, and local-service apps, start with:

- information hierarchy;
- content density;
- window width and posture;
- confirmation and save boundaries;
- clear layouts across phone, tablet, foldable, and multi-window states.

Use immersive-light components for high-value interaction surfaces. Reserve 3DGS or heavier spatial content for genuine 3D scenarios such as spatial reconstruction, product presentation, cultural-tourism exhibits, or other content that benefits from a spatial model.

## Page Prioritization

Before changing a screen, score it by frequency, information complexity, and visual-feedback needs:

| Question | Prioritize when | Defer when |
| --- | --- | --- |
| Usage frequency | Home, workbench, list, detail | Low-frequency settings or one-off pages |
| Information complexity | Multiple modules, states, or data levels | One control or one field |
| Interaction feedback | Drag, select, preview, confirm, or status changes | Static explanation or simple redirect |

The first adaptation pass should usually cover these five page groups:

1. Home and workbench: establish information priority and primary actions.
2. List and detail: use a master-detail relationship on wide windows.
3. Edit and form: make source, draft, user edits, validation, and save state explicit.
4. Empty and guidance states: provide a clear next action instead of decorative space.
5. Settings and management: favor stable grouping, permissions, data, and risky actions.

Do not force every screen into spatial treatment. A login page or a simple toggle page often benefits from less motion and fewer layers.

## Window And Posture Strategy

Treat the current window width and device posture as layout inputs, not as page-specific magic numbers. Feed them from the project's window/device abstraction and keep the breakpoint policy in one place.

```ts
const PAGE_PADDING = 24;
const LIST_WIDTH = 320;
const DETAIL_MIN_WIDTH = 560;
const SPLIT_GAP = 16;

function canUseMasterDetail(windowWidth: number): boolean {
  const availableWidth = windowWidth - PAGE_PADDING * 2;
  return availableWidth >= LIST_WIDTH + SPLIT_GAP + DETAIL_MIN_WIDTH;
}
```

These values are starting points, not HarmonyOS constants. Replace them with the project's design tokens and measure the actual usable window area after safe-area and window-inset handling.

Recommended first-pass behavior:

| State | Layout direction |
| --- | --- |
| Phone portrait | Single column; primary task first |
| Phone landscape | Single column or restrained two-column layout |
| Foldable expanded | Primary tasks on the left, supporting information on the right |
| Tablet landscape | Workbench sections or list-detail split |
| Multi-window | Preserve readable card widths; hide low-priority decoration |

Do not simply reveal every hidden module on a large screen. More space should clarify relationships, not increase cognitive load.

## Home And Workbench

Model workbench sections with explicit priority and compact/expanded visibility rather than scattering width checks through the page:

```ts
interface WorkbenchSection {
  id: string;
  title: string;
  priority: 'primary' | 'secondary' | 'support';
  visibleInCompact: boolean;
  visibleInExpanded: boolean;
}
```

Typical sections include current tasks, recent records, reminders, project entry points, and shortcuts. On a phone, use a vertical order that puts the current task first. On an expanded device, place primary work on one side and recent/supporting information on the other. Keep the primary action visible in every state.

## List And Detail

Use a separate detail destination on a narrow window and a master-detail layout on a wide window:

- Keep the list selection as the source of truth for the detail content.
- Show an intentional empty-selection state in the detail pane, or select a sensible default item.
- Make the selected row obvious on the left side.
- Decide whether editing is local to the detail pane or opens a new route; do not let the behavior change accidentally at a breakpoint.
- Keep the back behavior predictable when switching between single-pane and split layouts.

This pattern applies to meetings, contacts, projects, files, messages, favorites, and local-service items.

## Edit And Form

Editing surfaces should optimize for input, validation, save, and confirmation before spatial decoration. Keep these states separate when generated content or AI assistance is involved:

```ts
interface DraftEditState {
  sourceText: string;
  generatedDraft: string;
  userEditedContent: string;
  hasUserConfirmed: boolean;
  hasUnsavedChanges: boolean;
}
```

On large screens, a useful pattern is reference material on the left and the final editable content on the right. On phones, use a vertical flow. Keep the object being edited, unsaved status, validation errors, save, cancel, and confirmation boundaries visible and consistent across devices.

## Empty, Settings, And Low-Frequency Pages

An empty state should answer “what can I do next?” with one or two concrete actions, such as create, import, retry, clear filters, or view an example. An empty right detail pane should also provide a prompt rather than remaining blank.

Settings pages should stay calm and legible, especially for account, privacy, permissions, notifications, synchronization, backup, export, and destructive operations. On wide windows, use a category list on the left and the selected setting detail on the right; on phones, keep grouped navigation. Keep risky actions visually and procedurally distinct.

## Spatialization And Motion Rules

- Add spatial depth where it clarifies hierarchy, focus, selection, or continuity.
- Keep high-frequency operations fast and predictable; do not make every card float or morph.
- Use immersive light or one-shot transitions for focal interactions, then let surrounding content remain stable.
- Keep 3D or reconstruction assets out of ordinary forms and settings unless they are part of the actual task.
- Test reduced motion, font scaling, contrast, and long content before adding more visual layers.

## Validation Checklist

- Test phone portrait, phone landscape, tablet landscape, foldable compact/expanded postures, and multi-window widths.
- Verify that page hierarchy and primary actions remain clear after module reflow.
- Check list selection, detail empty state, back navigation, edit/save flow, and unsaved changes at every breakpoint.
- Confirm cards and controls have stable minimum widths; avoid clipped text and over-dense columns.
- Test safe areas, system bars, keyboard appearance, font scale, dark mode, accessibility contrast, and reduced motion.
- Measure real devices when possible; Preview alone does not prove foldable posture, window resizing, or material performance.

## Common Pitfalls

- Treating tablet support as a stretched phone screenshot.
- Adding 3DGS or heavy spatial effects before fixing information architecture.
- Revealing every module on large screens without priority or density rules.
- Leaving the detail pane blank when no list item is selected.
- Mixing source text, AI draft, and user-confirmed content in one edit state.
- Using scattered width checks that disagree across pages.
- Letting spatial motion obscure save, confirmation, permission, or destructive-action boundaries.
