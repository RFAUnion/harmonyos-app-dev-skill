# One-Shot And Shared-Element Transitions

Use this reference when the user asks for `一镜到底`, shared-element transitions, shared-container transitions, image/card/list/book/video expansion transitions, `geometryTransition`, `NodeController`, `NodeContainer`, `customNavContentTransition`, `NavigationAnimatedTransition`, or `componentSnapshot`.

Source best practice: `https://developer.huawei.com/consumer/cn/doc/best-practices/bpta-one-shot-to-the-end`

Markdown source: `https://developer.huawei.com/consumer/cn/doc/best-practices/bpta-one-shot-to-the-end.md`

## Intent

One-shot transition design connects a source element and a destination element during page or state changes. Use it when the user should perceive continuity between two screens or states: thumbnail to detail image, search box to search page, card/list item to detail page, book cover to reading page, or video card to full-screen playback.

Do not add this effect everywhere. Use it for focal elements that carry user intent across the transition. Keep default Navigation transitions for ordinary page changes.

## Choose The Implementation

Use `geometryTransition()` when:

- The source and destination are different ArkUI components that can be cheaply created.
- Both components can bind the same transition id.
- The state or route mutation can be placed inside `animateTo`.
- The destination can disable or reduce default Navigation animation when needed.

Use `NodeController` / `NodeContainer` migration when:

- Recreating the node is expensive or visually wrong.
- The content must continue running during transition, such as video or live playback.
- Gesture-driven expansion needs to move the same node between containers.
- You can calculate source and target geometry and animate scale/translation manually.

Use `Navigation.customNavContentTransition()` when:

- The transition belongs to page routing rather than only local component state.
- You need shared-container behavior: card to detail, book flip, video detail, or complex page-level choreography.
- You need a fallback to default Navigation transition when the custom transition cannot be prepared.
- Multiple `NavDestination` pages need to register/unregister transition handlers.

## Geometry Transition Pattern

Bind matching source and destination elements with a stable id:

```ts
Image($r('app.media.cover'))
  .geometryTransition(this.selectedId)

Column()
  .geometryTransition(this.param.geometryId)
```

Run the state or path-stack mutation inside `animateTo`:

```ts
this.getUIContext().animateTo({ duration: 250 }, () => {
  this.pathStack.pushPath({ name: 'DetailPage', param }, false);
});
```

The `false` animation flag is useful when the shared-element animation should not fight the default Navigation transition. Verify the exact `pushPath`/`pushPathByName` overload in the installed SDK.

For search boxes, avatars, and list rows, use the same transition id on both pages and add opacity/visibility transitions so only the shared element appears continuous while surrounding content fades normally.

## Node Migration Pattern

Use a `NodeController` that owns the reusable node and render it through `NodeContainer` in whichever page or overlay currently hosts it.

Practical flow:

1. Build the reusable node with `BuilderNode` in a custom `NodeController`.
2. Render it in the source page through `NodeContainer`.
3. On gesture or click, capture source geometry and prepare destination geometry.
4. Move the node to the destination container or overlay.
5. Animate translation, scale, clipping, and radius from source geometry to target geometry.
6. Reset or detach the temporary overlay after the transition completes.

Use this for video expansion when playback continuity matters. Avoid creating a second video component that restarts playback during the transition.

## Custom Navigation Transition Pattern

For route-level one-shot transitions, centralize registration so each page can safely opt in:

```ts
type TransitionRunner = (proxy: NavigationTransitionProxy) => void;

class CustomNavigationTransitions {
  private runners: Map<string, TransitionRunner> = new Map();

  register(id: string, runner: TransitionRunner): void {
    this.runners.set(id, runner);
  }

  unregister(id: string): void {
    this.runners.delete(id);
  }
}
```

Then wire `Navigation.customNavContentTransition()` to return a `NavigationAnimatedTransition` for supported page pairs and `undefined` for all other routes, so the system default transition remains available.

Inside a destination page:

- Read `NavDestinationContext` in `onReady`.
- Keep `pathStack` and `navDestinationId` if needed.
- Register the custom transition after the destination has enough layout/param information.
- Unregister in `onDisAppear` or equivalent cleanup.

Use `componentSnapshot()` for card/list source views when a direct shared node is awkward. Display the snapshot during the transition to avoid white flashes, then fade to the real detail page content.

## Scenario Notes

- Image pinch expansion: prefer node migration when gesture scale controls the transition and the image should become a dialog-like full-screen page.
- Thumbnail to gallery: prefer `geometryTransition()` with per-item stable ids and disabled default Navigation animation.
- Search/avatar expansion: prefer `geometryTransition(..., { follow: true })` when the shared component should follow its counterpart.
- Card to detail: prefer custom Navigation transition plus snapshot when the source card layout is complex or a WaterFlow item.
- List row to detail: `geometryTransition()` is often enough when the list item and destination container have simple matching shapes.
- Book flip: use custom Navigation transition and rotate/scale properties around the correct edge.
- Video expansion: prefer node migration plus custom Navigation transition to preserve playback state.

## Validation Checklist

- Confirm all shared ids are stable and unique for the active list/grid item.
- Ensure source and destination elements are not both visible in a way that causes double images.
- Disable default route animation when it conflicts with the one-shot animation.
- Verify back navigation; the reverse transition should either mirror the forward transition or deliberately fall back.
- Clean up registered custom transitions on page disappearance.
- Test interrupted transitions: rapid taps, back gesture during animation, rotation/window resize, and failed snapshot capture.
- Test on device, not only Preview, especially for video, WaterFlow, and gesture-driven transitions.

## Common Pitfalls

- Using `geometryTransition()` without wrapping the state change in `animateTo`.
- Reusing the same transition id for multiple visible list items.
- Forgetting to hide the clicked source item while a snapshot or destination copy is animating.
- Letting Navigation's default transition run at the same time as a custom shared-element transition.
- Recreating expensive nodes such as video players instead of migrating the existing node.
- Failing to unregister custom transition handlers, leaving stale page ids in memory.
