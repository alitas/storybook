---
title: 'Parameters'
---

Parameters are static metadata used to configure your [stories](../get-started/whats-a-story.md) and [addons](../addons/introduction.md) in Storybook. They are specified at the story, meta (component), project (global) levels.

## Story parameters

<div class="aside">

ℹ️ Parameters specified at the story level will [override](#parameter-inheritance) those specified at the project level and meta (component) level.

</div>

Parameters specified at the story level apply to that story only. They are defined in the `parameters` property of the story (named export):

<!-- prettier-ignore-start -->

<CodeSnippets
  paths={[
    'angular/parameters-in-story.ts.mdx',
    'web-components/parameters-in-story.js.mdx',
    'web-components/parameters-in-story.ts.mdx',
    'common/parameters-in-story.js.mdx',
    'common/parameters-in-story.ts.mdx',
  ]}
/>

<!-- prettier-ignore-end -->

## Meta parameters

<div class="aside">

ℹ️ Parameters specified at the meta (component) level will [override](#parameter-inheritance) those specified at the project level.

</div>

Parameter's specified in a [CSF](../writing-stories/introduction.md#component-story-format-csf) file's meta configuration apply to all stories in that file. They are defined in the `parameters` property of the `meta` (default export):

<!-- prettier-ignore-start -->

<CodeSnippets
  paths={[
    'angular/parameters-in-meta.ts.mdx',
    'web-components/parameters-in-meta.js.mdx',
    'web-components/parameters-in-meta.ts.mdx',
    'common/parameters-in-meta.js.mdx',
    'common/parameters-in-meta.ts.mdx',
  ]}
/>

<!-- prettier-ignore-end -->

## Project parameters

Parameters specified at the project (global) level apply to **all stories** in your Storybook. They are defined in the `parameters` property of the default export in your `.storybook/preview.js|ts` file:

<!-- prettier-ignore-start -->

<CodeSnippets
  paths={[
   'common/parameters-in-preview.js.mdx',
   'common/parameters-in-preview.ts.mdx',
  ]}
/>

<!-- prettier-ignore-end -->

## Available parameters

Storybook only accepts a few parameters directly.

### `layout`

Type: `'centered' | 'fullscreen' | 'padded'`

Default: `'padded'`

Specifies how the canvas should [lay out the story](../configure/story-layout.md).

- **centered**: Center the story within the canvas
- **padded**: (default) Add padding to the story
- **fullscreen**: Show the story as-is, without padding

### `options`

Type:

```ts
{
  storySort?: StorySortConfig | StorySortFn;
}
```

<Callout variant="warning">

The `options` parameter can _only_ be applied at the [project level](#project-parameters).

</Callout>

#### `options.storySort`

Type: `StorySortConfig | StorySortFn`

```ts
type StorySortConfig = {
  includeNames?: boolean;
  locales?: string;
  method?: 'alphabetical' | 'alphabetical-by-kind' | 'custom';
  order?: string[];
};

type Story = {
  id: string;
  importPath: string;
  name: string;
  title: string;
};

type StorySortFn = (a: Story, b: Story) => number;
```

Specifies the order in which stories are displayed in the Storybook UI.

When specifying a configuration object, the following options are available:

- **includeNames**: Whether to include the story name in the sorting algorithm. Defaults to `false`.
- **locales**: The locale to use when sorting stories. Defaults to your system locale.
- **method**: The sorting method to use. Defaults to `alphabetical`.
  - **alphabetical**: Sort stories alphabetically by name.
  - **alphabetical-by-kind**: Sort stories alphabetically by kind, then by name.
  - **custom**: Use a custom sorting function.
- **order**: Stories in the specified order will be displayed first, in the order specified. All other stories will be displayed after, in alphabetical order. The order array can accept a nested array to sort 2nd-level story kinds, e.g. `['Intro', 'Pages', ['Home', 'Login', 'Admin'], 'Components']`.

When specifying a custom sorting function, the function behaves like a typical JavaScript sorting function. It accepts two stories to compare and returns a number. For example:

```js
(a, b) => (a.id === b.id ? 0 : a.id.localeCompare(b.id, undefined, { numeric: true }));
```

See [the guide](../writing-stories/naming-components-and-hierarchy/#sorting-stories) for usage examples.

---

### Essential addons

All other parameters are contributed by addons. The [essential addon's](../addons/essentials.md) parameters are documented on their individual pages:

- [Actions](../essentials/actions.md#parameters)
- [Backgrounds](../essentials/backgrounds.md#parameters)
- [Controls](../essentials/controls.md#parameters)
- [Highlight](../essentials/highlight.md#parameters)
- [Interactions](../essentials/interactions.md#parameters)
- [Measure & Outline](../essentials/measure-and-outline.md#parameters)
- [Viewport](../essentials/viewport.md#parameters)

## Parameter inheritance

No matter where they're specified, parameters are ultimately applied to a single story. Parameters specified at the project (global) level are applied to every story in that project. Those specified at the meta (component) level are applied to every story associated with that meta. And parameters specified for a story only apply to that story.

When specifying parameters, they are merged together in order of increasing specificity:

1. Project (global) parameters
2. Meta (component) parameters
3. Story parameters

<div class="aside">

ℹ️ Parameters are **merged**, so objects are deep-merged, but arrays and other properties are overwritten.

</div>

In other words, the following specifications of parameters:

```js
// .storybook/preview.js|ts

const preview = {
  // 👇 Project-level parameters
  parameters: {
    layout: 'centered',
    demo: {
      demoProperty: 'a',
      demoArray: [1, 2],
    },
  },
  // ...
};
export default preview;
```

```js
// Dialog.stories.js|ts

const meta = {
  component: Dialog,
  // 👇 Meta-level parameters
  parameters: {
    layout: 'fullscreen',
    demo: {
      demoProperty: 'b',
      anotherDemoProperty: 'b',
    },
  },
};
export default meta;

// (no additional parameters specified)
export const Basic = {};

export const LargeScreen = {
  // 👇 Story-level parameters
  parameters: {
    layout: 'padded',
    demo: {
      demoArray: [3, 4],
    },
  },
};
```

Will result in the following parameter values applied to each story:

```js
// Applied story parameters

// For the Basic story:
{
  layout: 'fullscreen',
  demo: {
    demoProperty: 'b',
    anotherDemoProperty: 'b',
    demoArray: [1, 2],
  },
}

// For the LargeScreen story:
{
  layout: 'padded',
  demo: {
    demoProperty: 'b',
    anotherDemoProperty: 'b',
    demoArray: [3, 4],
  },
}
```
