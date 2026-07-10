# ArkTS And ArkUI

## ArkTS Rules Of Thumb

ArkTS is based on TypeScript but is stricter for stability and performance.

Prefer:

- Explicit static types for state, function parameters, and returned objects.
- Stable object shapes; avoid adding/removing arbitrary object fields at runtime.
- Clear classes/interfaces for app data models.
- `TaskPool` or `Worker` for expensive work instead of blocking UI/lifecycle callbacks.

Avoid:

- Using `any` unless forced by external APIs.
- Dynamic JS-style object mutation.
- High-frequency logging in UI update paths.
- Heavy work in `build()` or lifecycle callbacks.

## ArkUI Page Pattern

Minimal page:

```ts
@Entry
@Component
struct Index {
  @State message: string = 'Hello World';

  build() {
    RelativeContainer() {
      Text(this.message)
        .id('HelloWorld')
        .fontSize($r('app.float.page_text_font_size'))
        .fontWeight(FontWeight.Bold)
        .alignRules({
          center: { anchor: '__container__', align: VerticalAlign.Center },
          middle: { anchor: '__container__', align: HorizontalAlign.Center }
        })
        .onClick(() => {
          this.message = 'Welcome';
        })
    }
    .height('100%')
    .width('100%')
  }
}
```

## State And Rendering

Use state variables for values that drive UI updates. Keep derived display values small and deterministic.

For lists/grids:

- Use stable keys.
- Avoid stringifying large objects in key generators.
- Keep item components reusable when rendering many rows.
- Do not access or mutate large state repeatedly inside tight render loops.

## Navigation

For simple generated projects, `UIAbility` often loads `pages/Index` directly.

For multi-page apps, prefer ArkUI navigation patterns (`Navigation`, `NavDestination`) when the project is already structured that way. Keep page registration and `main_pages.json` consistent with the actual routing approach.

## Layout Guidance

Use ArkUI layout components intentionally:

- `Column` and `Row` for common linear layouts.
- `RelativeContainer` when anchoring is clearer than nesting.
- `List`, `Grid`, and lazy rendering for repeated content.
- Resource-based dimensions/colors/strings for maintainability.

Keep screens adaptive. Avoid hardcoding phone-only assumptions unless the module's `deviceTypes` and product requirements make that explicit.
