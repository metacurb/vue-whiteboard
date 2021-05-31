# vue-whiteboard
Built using Vue 3, based upon the fantastic [ng-whiteboard](https://github.com/mostafazke/ng-whiteboard)

This library has no styles. It simply acts as a wrapper for the basic whiteboard functionality.

## Lightweight Vue whiteboard

**Features:**

- Supports touch.
- Custom colors.
- Custom background colors.
- Custom stroke size.
- Export as PNG
- Undo & Redo

# Install

1. Install npm module:

  ```bash
  npm i @beauagst/vue-whiteboard
  ```

2. Add the module to your project

  ```javascript
  import VueWhiteboard from 'vue-whiteboard'
  ...

  export default {
    ...
    components: {
      VueWhiteboard
    }
    ...
  }
  ```

3. Insert the Whiteboard Component element into your template.

  ```html
  <template>
    <vue-whiteboard />
  </template>
  ```

## Props

| Input | Type | Default | Required | Description |
| --- | --- | --- | --- | --- |
| `color` | `string` | `#000000` | `false` | Set brush color |
| `backgroundColor`   | `string` | `#ffffff` | `false` | Set whiteboard background color |
| `size` | `string` | `5px` | `false` | Set brush size |
| `linejoin` | `string` | `round` | `false` | Define the shape of two lines when joined together (`'miter'` , `'round'` , `'bevel'` , `'miter-clip'` , `'arcs'`) |
| `linecap` | `string` | `round` | `false` | Define start and end shape of line (`'butt'`, `'square'`, `'round'`) |
| `lineStyles` | `Object` | `{}` | `false` | Any extra line styles you'd like to apply. Will be inlined with the rest. |

## Events

| Name | Description | Payload |
| --- | --- | --- |
| `init`  | On component initialization | d3 Selection |
| `undo`  | On undo last line | `undefined` |
| `redo`  | On repaint last line | `undefined` |
| `save`  | On save | `undefined` |
| `clear`  | On clear | `undefined` |
| `dragstart`  | Once dragging starts within SVG | `{ coordinates: [X, Y], node: Path }` |
| `drag` | Continuing dragging | `{ coordinates: [X, Y], node: Path }` |
| `dragend` | Once dragging has ended | `{ coordinates: [X, Y], node: Path }` |

## Methods
Methods can be accessed from the component.

```html
<template>
  <vue-whiteboard ref="whiteboard" />
</template>
```

```javascript
export default {
  ...
  methods: {
    save() {
      const res = await this.$refs.whiteboard.save()
    }
  }
}
```

| Name | Description |
| --- | --- |
| undo | Undo last line |
| redo | Repaint last line |
| save | Return base64-formatted PNG of whiteboard |
| clear | Clear the whiteboard |

## Contributing

The project is open for contributors! Please file an issue or make a PR :)
