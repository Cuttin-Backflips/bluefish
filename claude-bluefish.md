# Claude's Bluefish Quick-Start Guide

> Read this file instead of exploring the entire bluefish folder.
> It contains everything needed to create diagrams with the user.

## What Is Bluefish?

Bluefish is a **declarative diagramming framework** built on **SolidJS** that renders to **SVG**.
You compose diagrams from simple building blocks (shapes, layouts, arrows) using TSX.
The library lives at `packages/bluefish-solid/` in this monorepo.

- **Language**: TypeScript + SolidJS JSX (TSX)
- **Renders to**: SVG in the browser
- **Key principle**: "Local reasoning" -- each component can be understood in isolation
- **Version**: 0.0.38 (beta)

## How to Start the Dev Server

The monorepo already has dependencies installed. The playground is in `packages/bluefish-solid/`.

### One-time setup

Copy the template to create the playground file (this file is gitignored):

```
copy packages\bluefish-solid\public\App.template.tsx packages\bluefish-solid\public\App.tsx
```

### Start the server

```
cd packages\bluefish-solid
pnpm dev
```

This opens **http://localhost:3000** with hot-reload. Edit `public/App.tsx` and the browser updates.

### File that gets edited

**`packages/bluefish-solid/public/App.tsx`** -- this is our diagram playground.

The entry point `public/index.tsx` renders `<App />` into `<div id="root">` in `public/index.html`.

## Minimal App.tsx Template

```tsx
import { type Component } from "solid-js";
import { Bluefish, Rect, Text, Arrow, Ref, Group, StackH, StackV, Align, Background, Circle, Line, Distribute } from "../src";
import { createName } from "../src/createName";
import withBluefish, { WithBluefishProps } from "../src/withBluefish";

const App: Component = () => {
  return (
    <Bluefish>
      {/* diagram goes here */}
    </Bluefish>
  );
};

export default App;
```

All imports come from `"../src"` (relative to the `public/` folder, pointing at the library source).

---

## Component API Reference

### `<Bluefish>` -- Root container

Wraps everything. Renders an `<svg>` element.

```tsx
<Bluefish width={800} height={600} padding={10} debug={false}>
  {/* children */}
</Bluefish>
```

| Prop | Type | Default | Notes |
|------|------|---------|-------|
| `width` | `number?` | auto-fit | SVG width |
| `height` | `number?` | auto-fit | SVG height |
| `padding` | `number?` | `10` | Padding around content |
| `debug` | `boolean?` | `false` | Shows scenegraph JSON below diagram |

---

### `<Rect>` -- Rectangle

Standard SVG rect. Accepts all SVG rect attributes (fill, stroke, rx, ry, opacity, etc).

```tsx
<Rect name={myName} width={100} height={50} fill="cornflowerblue" stroke="black" rx={5} />
```

| Prop | Type | Required | Notes |
|------|------|----------|-------|
| `name` | `Id` | yes (auto if wrapped) | Unique identifier |
| `width` | `number?` | no | Width in px |
| `height` | `number?` | no | Height in px |
| `x`, `y` | `number?` | no | Position |
| + all SVG rect attrs | | | fill, stroke, rx, opacity, etc. |

---

### `<Circle>` -- Circle

```tsx
<Circle name={myName} r={25} fill="red" />
```

| Prop | Type | Required | Notes |
|------|------|----------|-------|
| `name` | `Id` | yes | Unique identifier |
| `r` | `number` | **yes** | Radius |
| `cx`, `cy` | `number?` | no | Center position |
| + all SVG circle attrs | | | fill, stroke, etc. |

---

### `<Text>` -- Text label

Renders SVG text with automatic bounding box calculation.

```tsx
<Text name={myName} font-size="18px" fill="black">Hello World</Text>
```

| Prop | Type | Default | Notes |
|------|------|---------|-------|
| `name` | `Id` | auto | Unique identifier |
| `children` | `string \| number` | | The text content |
| `font-family` | `string` | `"Alegreya Sans, sans-serif"` | |
| `font-size` | `string` | `"14"` | |
| `font-weight` | `number` | `700` | |
| `fill` | `string` | inherited | Text color |
| `width` | `number?` | | Max width (wraps words) |
| `dx`, `dy` | `string \| number` | | Offset |

---

### `<Arrow>` -- Smart arrow between two elements

Takes exactly **two `<Ref>` children**: source and target. Uses `perfect-arrows` for routing.

```tsx
<Arrow bow={0.2} stretch={0.5} padStart={5} padEnd={20} stroke="black" stroke-width={3}>
  <Ref select={sourceName} />
  <Ref select={targetName} />
</Arrow>
```

| Prop | Type | Default | Notes |
|------|------|---------|-------|
| `bow` | `number` | `0.2` | Curvature amount |
| `stretch` | `number` | `0.5` | How much the arrow stretches |
| `stretchMin` | `number` | `40` | Min stretch |
| `stretchMax` | `number` | `420` | Max stretch |
| `padStart` | `number` | `5` | Gap from source |
| `padEnd` | `number` | `20` | Gap from target (arrowhead) |
| `flip` | `boolean` | `false` | Flip curve direction |
| `straights` | `boolean` | `true` | Allow straight arrows |
| `start` | `boolean` | `false` | Show circle at start point |
| `stroke` | `string` | `"black"` | |
| `stroke-width` | `number` | `3` | |

---

### `<Line>` -- Straight line between two elements

Like Arrow but renders a simple line. Takes two `<Ref>` children.

```tsx
<Line stroke="gray" stroke-width={2} stroke-dasharray="5,5">
  <Ref select={fromName} />
  <Ref select={toName} />
</Line>
```

| Prop | Type | Default | Notes |
|------|------|---------|-------|
| `stroke` | `string` | `"black"` | |
| `stroke-width` | `number` | `3` | |
| `stroke-dasharray` | `string?` | | For dashed lines |
| `source` | `number[]?` | | `[x, y]` on source bbox (0-1 range) |
| `target` | `number[]?` | | `[x, y]` on target bbox (0-1 range) |

---

### `<Ref>` -- Reference to a named element

Used inside Arrow, Line, Align, Background, etc. to point at another element by name.

```tsx
<Ref select={someName} />           // simple reference
<Ref select={[parentName, "child"]} /> // nested path into a named subtree
```

| Prop | Type | Notes |
|------|------|-------|
| `select` | `Id \| [Id, ...string[]]` | Name or path to the target element |

---

### `<StackH>` -- Horizontal stack

Lays children out left-to-right with spacing. Aligns vertically.

```tsx
<StackH spacing={20} alignment="centerY">
  <Rect width={50} height={50} fill="red" />
  <Rect width={50} height={50} fill="blue" />
</StackH>
```

| Prop | Type | Default | Notes |
|------|------|---------|-------|
| `spacing` | `number?` | | Gap between children (required unless `total` given) |
| `alignment` | `"top" \| "centerY" \| "bottom"` | `"centerY"` | Vertical alignment |
| `total` | `number?` | | Total width to distribute across |

---

### `<StackV>` -- Vertical stack

Lays children out top-to-bottom with spacing. Aligns horizontally.

```tsx
<StackV spacing={20} alignment="centerX">
  <Rect width={50} height={50} fill="red" />
  <Rect width={50} height={50} fill="blue" />
</StackV>
```

| Prop | Type | Default | Notes |
|------|------|---------|-------|
| `spacing` | `number?` | | Gap between children |
| `alignment` | `"left" \| "centerX" \| "right"` | `"centerX"` | Horizontal alignment |
| `total` | `number?` | | Total height to distribute across |

---

### `<Align>` -- Align children to each other

Makes children share a position on one or two axes.

```tsx
<Align alignment="center">
  <Ref select={name1} />
  <Ref select={name2} />
</Align>
```

**Alignment values:**

| 1D | 2D |
|----|----|
| `"left"`, `"centerX"`, `"right"` | `"topLeft"`, `"topCenter"`, `"topRight"` |
| `"top"`, `"centerY"`, `"bottom"` | `"centerLeft"`, `"center"`, `"centerRight"` |
| | `"bottomLeft"`, `"bottomCenter"`, `"bottomRight"` |

---

### `<Group>` -- Container / grouping

Combines children's bounding boxes. Use as a wrapper for custom components.

```tsx
<Group>
  <Rect name={boxName} width={100} height={50} fill="blue" />
  <Text name={labelName}>Hello</Text>
  <Align alignment="center">
    <Ref select={boxName} />
    <Ref select={labelName} />
  </Align>
</Group>
```

| Prop | Type | Notes |
|------|------|-------|
| `x`, `y` | `number?` | Position the group |
| `rels` | `() => JSX.Element` | Additional relationship elements |

---

### `<Background>` -- Background shape around children

Wraps its `<Ref>` children with a background shape (default: stroked rect). The **first child** is the background shape; remaining children define the content area.

```tsx
<Background padding={10} fill="lightyellow" stroke="black" rx={8}>
  <Ref select={child1} />
  <Ref select={child2} />
</Background>
```

For custom background shapes:
```tsx
<Background padding={10} background={() => <Rect fill="none" stroke="teal" stroke-dasharray="5" rx={8} />}>
  {/* content children */}
</Background>
```

| Prop | Type | Default | Notes |
|------|------|---------|-------|
| `padding` | `number` | `10` | Padding around content |
| `background` | `() => JSX.Element` | stroked rect | Custom background shape |
| + SVG rect attrs | | | Applied to default rect background |

---

### `<Distribute>` -- Even distribution

Distributes children evenly in a direction. Must provide `spacing` and/or `total`.

```tsx
<Distribute direction="horizontal" spacing={20}>
  {/* children */}
</Distribute>
```

| Prop | Type | Notes |
|------|------|-------|
| `direction` | `"vertical" \| "horizontal"` | Required |
| `spacing` | `number?` | Gap between items |
| `total` | `number?` | Total extent to fill |

---

## Key Utilities

### `createName(label: string): Id`

Creates a unique scoped name for an element. **Must be called inside a component wrapped with `withBluefish`** (it uses SolidJS context).

```tsx
import { createName } from "../src/createName";

const myRect = createName("myRect");
const myLabel = createName("myLabel");
```

Names are used to:
1. Assign to a component's `name` prop
2. Reference via `<Ref select={name} />`

### `withBluefish(component, options?)`

Higher-order component that integrates a component into the Bluefish layout system. All custom components that participate in layout must be wrapped.

```tsx
import withBluefish, { WithBluefishProps } from "../src/withBluefish";

type MyNodeProps = WithBluefishProps<{
  label: string;
  color: string;
}>;

const MyNode = withBluefish((props: MyNodeProps) => {
  const boxName = createName("box");
  const textName = createName("text");

  return (
    <Group>
      <Rect name={boxName} width={80} height={40} fill={props.color} rx={5} />
      <Text name={textName}>{props.label}</Text>
      <Align alignment="center">
        <Ref select={boxName} />
        <Ref select={textName} />
      </Align>
    </Group>
  );
});
```

The `name` prop is automatically handled by `withBluefish` -- it becomes optional for callers but is injected internally.

---

## SolidJS Essentials (Not React!)

Bluefish uses **SolidJS**, not React. Key differences:

- **`<For each={array}>`** instead of `.map()` for lists:
  ```tsx
  <For each={items}>{(item, index) => <Rect />}</For>
  ```
  Note: `index` is an `Accessor` -- call it as `index()` to get the value.

- **`<Show when={condition}>`** instead of `{condition && ...}`:
  ```tsx
  <Show when={showArrow}>
    <Arrow>...</Arrow>
  </Show>
  ```

- **Props are NOT destructured**. Access as `props.x`, never `const { x } = props`.

- **`mergeProps(defaults, props)`** for default values (instead of destructuring defaults).

- **Signals** use `createSignal()` -- read with `value()`, write with `setValue()`.

---

## Complete Example: Tree Diagram

This is the canonical example showing the key patterns. See full source at:
`packages/bluefish-solid/src/stories/SimpleTree.stories.tsx`

```tsx
import { For, mergeProps } from "solid-js";
import { Bluefish, Group, Rect, Text, StackV, StackH, Arrow, Align, Ref } from "../src";
import { createName } from "../src/createName";
import withBluefish, { WithBluefishProps } from "../src/withBluefish";

// Custom node component
type NodeProps = WithBluefishProps<{ value: string }>;

const Node = withBluefish((props: NodeProps) => {
  props = mergeProps({ value: "?" }, props);
  const valueName = createName("value");
  const outlineName = createName("outline");

  return (
    <Group>
      <Rect name={outlineName} width={50} height={65} rx={5}
            fill="cornflowerblue" stroke="black" opacity={0.5} />
      <Text name={valueName} font-size="24px">{props.value}</Text>
      <Align alignment="centerY">
        <Ref select={valueName} />
        <Ref select={outlineName} />
      </Align>
      <Align alignment="centerX">
        <Ref select={valueName} />
        <Ref select={outlineName} />
      </Align>
    </Group>
  );
});

// Recursive tree component
type TreeData = { value?: string; subtrees?: TreeData[] };
type TreeProps = WithBluefishProps<TreeData>;

const Tree = withBluefish((props: TreeProps) => {
  props = mergeProps({ subtrees: [] as TreeData[], value: "?" }, props);
  const nodeName = createName("node");
  const subtreeNames = (props.subtrees ?? []).map((_, i) => createName(`subtree${i}`));

  return (
    <Group>
      <Node name={nodeName} value={props.value} />
      {props.subtrees?.length ? (
        <>
          <StackV spacing={50} alignment="centerX">
            <Ref select={nodeName} />
            <StackH alignment="centerY" spacing={50}>
              <For each={props.subtrees}>
                {(child, i) => <Tree name={subtreeNames[i()]} {...child} />}
              </For>
            </StackH>
          </StackV>
          <For each={props.subtrees}>
            {(child, i) => (
              <Arrow bow={0} stretch={0.1} flip>
                <Ref select={nodeName} />
                <Ref select={[subtreeNames[i()], "node"]} />
              </Arrow>
            )}
          </For>
        </>
      ) : null}
    </Group>
  );
});

// Usage in App.tsx:
const App = () => (
  <Bluefish>
    <Tree
      value="A"
      subtrees={[
        { value: "B", subtrees: [{ value: "D" }, { value: "E" }] },
        { value: "C", subtrees: [{ value: "F" }, { value: "G" }] },
      ]}
    />
  </Bluefish>
);
```

---

## Common Patterns

### Labeled box
```tsx
const boxName = createName("box");
const labelName = createName("label");

<Group>
  <Rect name={boxName} width={100} height={50} fill="lightblue" />
  <Text name={labelName}>Label</Text>
  <Align alignment="center">
    <Ref select={labelName} />
    <Ref select={boxName} />
  </Align>
</Group>
```

### Arrow between named elements
```tsx
<Arrow>
  <Ref select={fromName} />
  <Ref select={toName} />
</Arrow>
```

### Nested Ref into a named child of a component
```tsx
// If Tree has a child named "node":
<Ref select={[treeName, "node"]} />
```

### Custom background wrapper
```tsx
<Background padding={8} background={() => <Rect fill="none" stroke="teal" stroke-dasharray="8" rx={10} />}>
  <Ref select={child1} />
  <Ref select={child2} />
</Background>
```

---

## Files to Review for Deeper Understanding

If you need more detail on a specific component, read these source files:

| Topic | File |
|-------|------|
| All exports | `packages/bluefish-solid/src/index.ts` |
| Root Bluefish component | `packages/bluefish-solid/src/bluefish.tsx` |
| Rect | `packages/bluefish-solid/src/rect.tsx` |
| Circle | `packages/bluefish-solid/src/circle.tsx` |
| Text + types | `packages/bluefish-solid/src/text.tsx` and `src/text/types.ts` |
| Arrow | `packages/bluefish-solid/src/arrow.tsx` |
| Align + alignment types | `packages/bluefish-solid/src/align.tsx` |
| StackH / StackV | `packages/bluefish-solid/src/stackh.tsx`, `src/stackv.tsx`, `src/stackLayout.ts` |
| Group | `packages/bluefish-solid/src/group.tsx` |
| Ref | `packages/bluefish-solid/src/ref.tsx` |
| Background | `packages/bluefish-solid/src/background.tsx` |
| Line | `packages/bluefish-solid/src/line.tsx` |
| Distribute | `packages/bluefish-solid/src/distribute.tsx` |
| withBluefish HOC | `packages/bluefish-solid/src/withBluefish.tsx` |
| createName | `packages/bluefish-solid/src/createName.tsx` |
| Layout engine core | `packages/bluefish-solid/src/layout.tsx` |
| Scenegraph | `packages/bluefish-solid/src/scenegraph.ts` |

### Example files (complete working diagrams)

| Example | File |
|---------|------|
| Tree diagram | `packages/bluefish-solid/src/stories/SimpleTree.stories.tsx` |
| Insertion sort viz | `packages/bluefish-solid/src/example-gallery/insertion-sort.tsx` |
| Venn diagram | `packages/bluefish-solid/src/stories/VennDiagram.stories.tsx` |
| Recipe diagram | `packages/bluefish-solid/src/example-gallery/brownie.tsx` |
| File system diagram | `packages/bluefish-solid/src/example-gallery/DFSCQ-log-figure.tsx` |

---

## Project Structure for New Diagram Projects

For standalone projects (outside the monorepo), use this structure:

```
my-diagrams/
  package.json          # solidjs + bluefish-solid deps
  vite.config.ts        # SolidJS vite plugin
  tsconfig.json
  index.html            # Entry point
  src/
    App.tsx             # Main diagram
    index.tsx           # Render entry
```

But for our iterative workflow, we simply edit **`packages/bluefish-solid/public/App.tsx`**
within the existing monorepo. It's faster (no install step) and has direct access to source.

---

## Troubleshooting

- **"Could not find X"**: The `select` name in a `<Ref>` doesn't match any `name` prop. Check spelling and that `createName()` was called in the right scope.
- **Element has no size**: Shapes like `<Rect>` need explicit `width`/`height`. `<Text>` auto-calculates size.
- **Stack errors about unowned dimensions**: Every child in a `<StackH>` must have a known width; every child in `<StackV>` must have a known height.
- **Names must be created inside withBluefish**: `createName()` uses SolidJS context that only exists inside components wrapped with `withBluefish`.
- **Don't destructure props**: SolidJS props are reactive proxies. Always use `props.x`, never `const { x } = props`.
