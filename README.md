<!--
Get your module up and running quickly.

Find and replace all on all files (CMD+SHIFT+F):
- Name: Nuxt Countdown
- Package name: nuxt-countdown
- Description: Countdown component for Nuxt 3.
-->

# Nuxt Countdown

[![npm version][npm-version-src]][npm-version-href]
[![npm downloads][npm-downloads-src]][npm-downloads-href]
[![License][license-src]][license-href]
[![Nuxt][nuxt-src]][nuxt-href]

A countdown module for multi purpose usage written for Nuxt 3 or newer.

<!-- - [✨ &nbsp;Release Notes](/CHANGELOG.md) -->
<!-- - [🏀 Online playground](https://stackblitz.com/github/your-org/nuxt-countdown?file=playground%2Fapp.vue) -->
<!-- - [📖 &nbsp;Documentation](https://example.com) -->

## Features

<!-- Highlight some of the features your module provide here -->
- Nuxt 3 Ready
- Autoimport
- Less config

## Documentation

While we're preparing detailed documentation and playground, you can check the examples below.

## Quick Setup

1. Add `nuxt-countdown` dependency to your project

```bash
# Using yarn
yarn add nuxt-countdown

# Using npm
npm install nuxt-countdown

# Using pnpm
pnpm add nuxt-countdown
```

2. Add `nuxt-countdown` to the `modules` section of `nuxt.config.ts`

```js
export default defineNuxtConfig({
  modules: [
    'nuxt-countdown'
  ]
})
```

That's it! You can now use Countdown Component in your Nuxt app ✨

## Nuxt 3 Examples

When you add this module, the ```<Countdown>``` component will be **automatically imported** into the project.

### Basic Usage with Date Object

```js
<template>
  <Countdown
      v-slot="{ days, hours, minutes, seconds }"
      :date="new Date('Oct 19, 2026 16:50:30')"
    >
      Time Remaining: {{ days }} days, {{ hours }} hours,
      {{ minutes }} minutes, {{ seconds }} seconds.
  </Countdown>
</template>
```

### Basic Usage with Time in Milliseconds

```js
<template>
  <Countdown
    :time="2 * 24 * 60 * 60 * 1000"
    v-slot="{ days, hours, minutes, seconds }"
  >
    Time Remaining：{{ days }} days, {{ hours }} hours, {{ minutes }} minutes, {{ seconds }} seconds.
  </Countdown>
</template>
```

### Custom Interval

You can set custom interval time value for update countdown values.

```js
<template>
  <Countdown
    :time="time"
    :interval="100"
    v-slot="{ days, hours, minutes, seconds, milliseconds }"
  >
    New Year Countdown：{{ days }} days, {{ hours }} hours,
    {{ minutes }} minutes, {{ seconds }}.{{ Math.floor(milliseconds / 100) }}
    seconds.
  </Countdown>
</template>

<script setup lang="ts">
const now: Date = new Date();
const newYear: Date = new Date(now.getFullYear() + 1, 0, 1);
const time: Ref<number> = ref(newYear.getTime() - now.getTime());
<script />
```

### withYears Prop

If you want to show years in countdown, you can use ```withYear``` prop.
Default value of ```withYear``` prop is ```false```. You can set it to ```true``` to show years.

```js
<template>
<Countdown
    v-slot="{ years, days, hours, minutes, seconds }"
    :date="new Date('Oct 19, 2026 16:50:30')"
  >
    Time Remaining: {{ years }} years, {{ days }} days, {{ hours }} hours,
    {{ minutes }} minutes, {{ seconds }} seconds.
  </Countdown>
</template>
```

### Transform Slot Props

You can modify the slot props provided from component for different purposes with ```:transform``` prop. 

For example, following code adding 0 in front of the slot values if they are one digit:

```js
<template>
 <Countdown
    :time="2 * 24 * 60 * 60 * 1000"
    :transform="transformSlotProps"
    v-slot="{ days, hours, minutes, seconds }"
  >
    Time Remaining：{{ days }} days, {{ hours }} hours, {{ minutes }} minutes,
    {{ seconds }} seconds.
  </Countdown>
</template>

<script setup lang="ts">
const transformSlotProps = (props: Record<string, number>) => {
  const formattedProps: Record<string, string> = {};

  Object.entries(props).forEach(([key, value]) => {
    formattedProps[key] = (value as number) < 10 ? `0${value}` : String(value);
  });

  return formattedProps;
};
<script />
```

### Tag Prop

You can specify the wrapper element to render with ```:tag``` prop. Default value is ```div```.

```js
<template>
 <Countdown
    tag="span"
    :time="2 * 24 * 60 * 60 * 1000"
    v-slot="{ days, hours, minutes, seconds }"
  >
    Time Remaining：{{ days }} days, {{ hours }} hours, {{ minutes }} minutes,
    {{ seconds }} seconds.
  </Countdown>
</template>
```

Will render as: 

```html
<span>
  Time Remaining: 1 days, 23 hours, 59 minutes, 53 seconds. 
</span>
```

### Countdown On Demand

You might want to start countdown after some functionality. 

For example, the following code shows how to make disable a button after first click and adding a 60 second countdown until re-enable button. 

```js
<template>
  <button type="button" :disabled="counting" @click="startCountdown">
    <Countdown
      v-if="counting"
      v-slot="{ totalSeconds }"
      :time="60100" 
      @end="onCountdownEnd"
    >
      Fetch again {{ totalSeconds }} seconds later
    </Countdown>
    <span v-else>Fetch Verification Code</span>
  </button>
</template>

<script setup lang="ts">
const counting: Ref<boolean> = ref(false);

const startCountdown = () => {
  counting.value = true;
};

const onCountdownEnd = () => {
  counting.value = false;
};
<script />
```

### Total Time Remaining

```js
<template>
  <Countdown
    v-slot="{ totalDays, totalHours, totalMinutes, totalSeconds }"
    :date="new Date('Oct 19, 2026 16:50:30')"
  >
    <br />

    Time Remaining: {{ totalDays }} days or {{ totalHours }} hours or
    {{ totalMinutes }} minutes or {{ totalSeconds }} seconds.
  </Countdown>
</template>
```


## Module Options

With module options, you can set prefix countdown component. You must add `countdown` key to `nuxt.config.ts`, here's the example:

```js
export default defineNuxtConfig({
  modules: [
    'nuxt-countdown'
  ],

  countdown: {
    prefix: 'MY' // if it's not defined, you can use the components as shown as in the docs. 
  }
})
```

With prefix option, then you can use the components as:

```js
<template>
  <MYCountdown />
</template>
```

## Development

```bash
# Install dependencies
npm install

# Generate type stubs
npm run dev:prepare

# Develop with the playground
npm run dev

# Build the playground
npm run dev:build

# Run ESLint
npm run lint

# Release new version
npm run release
```
---

This software is licensed under the [MIT License](https://github.com/volkanakkus/nuxt-countdown/blob/main/LICENSE) | @volkanakkus | Special thanks to [@fengyuanchen](https://github.com/fengyuanchen) 💚 

<!-- Badges -->
[npm-version-src]: https://img.shields.io/npm/v/nuxt-countdown/latest.svg?style=flat&colorA=020420&colorB=00DC82
[npm-version-href]: https://npmjs.com/package/nuxt-countdown

[npm-downloads-src]: https://img.shields.io/npm/dm/nuxt-countdown.svg?style=flat&colorA=020420&colorB=00DC82
[npm-downloads-href]: https://npmjs.com/package/nuxt-countdown

[license-src]: https://img.shields.io/npm/l/nuxt-countdown.svg?style=flat&colorA=020420&colorB=00DC82
[license-href]: https://npmjs.com/package/nuxt-countdown

[nuxt-src]: https://img.shields.io/badge/Nuxt-020420?logo=nuxt.js
[nuxt-href]: https://nuxt.com
