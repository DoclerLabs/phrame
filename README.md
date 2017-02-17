# Phrame
Modular, configurable SASS framework mostly to be used with CSS modules.

# What **Phrame** offers:
- 0kb footprint
- Multilevel configuration system
- Config references
- Color/Scheme management
- Exposed breakpoint rules
- Easing library
- Helper mixins
- Modules
- No bloat
- Use on existing codebases

# Basic setup

```bash
npm install phrame2 --save
```

Now use it:

```scss
// Load the environment (has not output)
@import '~phrame2/core';

// Just a simple reset (optional)
@import '~phrame2/module/reset';

// Some basic styles (optional)
@import '~phrame2/module/basic';

// Expose breakpoints (optional)
@import '~phrame2/module/expose';
```

> **Phrame** related stuff (mixins, functions, etc.) are prefixed with an `_` so **Phrame** won't conflict your codebase.

---

# Config system

The config system uses Sass Maps as a multi dimensional array - since Sass is only able to handle a
single level (looking at you map-get).

With building **Phrame** my primary goal was to build a flexible tool to handle multiple
sites from one codebase. For this to work we needed a system where it's simple to override the common stuff
and makes easy to switch color schemes for example. To achieve this we've built a flow where you can overwrite
the default configuration.

## Flow

To use such behavior you need to import your own instance of **Phrame** into your codebase.

`Load Phrame Env -> Config Overrides -> Output`

Here's a basic structure with the ability to override config values:

```

├── components
│   └── MyAwsomeCmp
│        └── MyAwsomeCmp.js
│        └── MyAwsomeCmp.scss
│
└── scss
    ├── core.scss
    └── config.scss

```

With these contents (and flow backwords to the root):

`MyAwsomeCmp.js`
```js
import styles from './MyAwesomeCmp.scss'
```

`MyAwsomeCmp.scss`
```scss
// Always import this `core` instead of Phrame directly
@import '../../scss/core';

body {
    background-color: _scheme('body.bg');
    font-size: _config('size.large');

    @include _on('mobile') {
        background-color: _scheme('body.bg-on-mobile');
        font-size: _config('size.sub-sizes.small');
    }
}
```

`core.scss`
```scss
@import '~phrame2/core';
@import './config';
```

`config.scss`
```scss
// $_ is just a temporary assignment
// because this is how functions work in Sass
$_: _config('size', (
    'small': 3.0em,
    'big': 8.0em,
    'large': 1.4em,
    'sub-sizes': (
        'small': 1.0em,
        'big': 2.0em,
        'large': 3.4em
    )
));
```

# Config Management
In case you're building a multi-(site/color) solution, or just need to make
the environment configurable, you may end up having multiple configuration files.

**Phrame** comes with multiple solutions to easier manage these options/values.

## `@function _config(key[, value])`

- `key` The key of the config property (size.small)
- `value` (optional): any

> **Good to know:** key is optional too. Returns full config object. Good for debugging purposes.

> **Good to know:** this is just a shorthand function, `_config-set` and `_config-get` is available too.

### Usage examples

#### Override config
```scss
// Defaults
$_: _config('_namespace', (
    'option-1': 'original-value-1',
    'option-2': 'original-value-2',
    'option-3': 'original-value-3'
));

// Overwrites
$_: _config('_namespace', (
    'option-1': 'new-value-1',
    'option-new-1': 'new-value-2'
));

// Results
$_: _config('_namespace', (
    'option-1': 'new-value-1',
    'option-2': 'original-value-2',
    'option-3': 'original-value-3',
    'option-new-1': 'new-value-2'
));
```

#### Merge multiple levels
```scss
// Defaults
$_: _config('_namespace', (
    'option-1': (
        'inner-option-1': 12px
    ),
    'option-2': 'original-value-2'
));

// Overwrites
$_: _config('_namespace', (
    'option-1': (
        'new-inner-option': 'value'
    ),
    'option-2': 'original-value-2'
));

// Results
$_: _config('_namespace', (
    'option-1': (
        'inner-option-1': 12px,
        'new-inner-option': 'value'
    ),
    'option-2': 'original-value-2'
));
```

#### Simplified write
```scss
// Defaults
$_: _config('_namespace', (
    'option-1': (
        'inner-option': 12px
    ),
    'option-2': 'original-value-2'
));

// Set new value
$_: _config('_namespace.option-1.inner-option', 20px);

// Results
$_: _config('_namespace', (
    'option-1': (
        'inner-option': 20px
    ),
    'option-2': 'original-value-2'
));
```

## Built-in Tools

There are some built-in utilities based on the config management system.
All of them have shorthands, but under the hood the're using `_config()`
with a namespace like `_scheme`.

These tools are made to manage the most common aspects:

- **path:** asset urls
- **scheme:** colors
- **screen:** responsiveness
- **typography:** sizes, families

### `_path($key[, $filename])`

- `$key` The key of the path
- `$filename` _(optional)_ Name of the file

```scss
// Defaults from Phrame
$_: _config('_path', (
    'backgrounds': '../backgrounds/',
));

// Extend with your own
$_: _path('_path', (
    'medals': (
        'direct-bg': '../images/medals/bg.png',
        'bronze': '../images/medals/bronze/',
        'silver': '../images/medals/silver/',
        'gold': '../images/medals/gold/'
    )
));

// Use it
body {
    background-image: url(_path('backgrounds', 'blue.jpg'));
}

.medals{
    background-image: url(_path('medals.direct-bg'));
}

.medal {
    &-gold {
        background-image: url(_path('medals.gold', 'small.jpg'));
    }
    ...
}
```

### `_scheme($key[, $value])`

- `$key` Scheme key path
- `$value` _(optional)_ Color library reference or unit value

This is a tricky stuff, because built-in color management splits into 2 part:
1. Color library
2. Scheme

The **Color library** is responsibe to store the colors:

```scss
$_color-library: (
    'transparent': transparent,
    'white': (
        'main': #ffffff
    ),
    'black': (
        'main': #000000
    ),
    'red': (
        'main': #ff0000
    ),
    'yellow': (
        'main': #ffcc00
    ),
    'green': (
        'main': #00ff00
    ),
    'social': (
        'facebook': #3B5998,
        'twitter': #55ACEE,
        'instagram': #3F729B,
        'google-plus': #DD4B39,
        'tumblr': #36465D
    ),
    'gradient': (
        'white-to-black': (#ffffff #000000)
    )
);
```

Then use **Scheme** to define the colors for your elements.

```scss
$_scheme-config: (

    // Commons
    common: (
        alert: 'yellow.main',
        success: 'green.main',
        error: 'red.main',
        disabled: 'gray.main'
    ),

    // Typography
    text: (
        base: 'black.main',
        link: 'yellow.main',
        link-hover: 'black.main'
    )
);
```

The use it in your stylesheet:

```scss
.alert {
    background-color: _scheme('common.alert');
}
```

What's the problem with using 1 level, or just variables?

Sure you can do the following in case you don't need such complexity.

```scss
$_: _config('colors', (
    'main': #000,
    'secondary': #fff
));

body {
    color: _config('colors.main');
}
```

At a certian point you'll end up having the same color values repeated
dozens of times. Keeping the available color palette in the **Color Library**
solves this issue, helps you prevent duplicates and also provides a proper name
(key) to get the correct color.

### `_color-library($key[, $value])`

- `$key` Key of the value
- `$value` _(optional)_ Value itself

Shorthand function to get/set color-library related values.

### `_screen($key[, $value])`

- `$key` Key of the value
- `$value` _(optional)_ Value itself

Shorthand function to get/set screen related values.

These values will be used by the built-in `_on` mixin.

```scss
// Defaults
$_screen-config: (
    size: (
        mobile: (
            from: 0,
            to: 700
        ),
        tablet: (
            from: 701,
            to: 1024,
        ),
        small: (
            from: 1025,
            to: 1440
        ),
        medium: (
            from: 1441,
            to: 1920,
        ),
        large: (
            from: 1921,
            to: 3000
        )
    )
);

// Add a new screen size
$_: _screen('size.myDevice', (
    from: 0,
    to: 3000
));

// Or modify
$_: _screen('size', (
    mobile: (
        from: 0,
        to: 625
    )
));
```

### `_typo($key[, $value])`

- `$key` Key of the value
- `$value` _(optional)_ Value itself

Shorthand function to get/set typography related values.

These values are used by the `basic` module and stores
your typography related values.

```scss
// Defaults
$_typo-config: (
    font-size: (
        root: 62.5%, // Should not be changed, used for em calculations.
        base: 14
    ),
    line-height: (
        root: 1.5
    ),
    font-family: (
        sans: unquote('Arial, Helvetica, sans-serif'),
        serif: unquote('"Times New Roman", Times, serif'),
        secondary: unquote('"Courier New", Courier, monospace')
    )
);

pre, code {
    font-family: _typo('font-family.secondary');
}
```

These values will be used by the `basic` module, but it's optional.

See `basic` module for more details.

### `_em($target, $base)`

Function used for `em` value calculations.

```scss
div {
    font-size: 1.4em; // 14px
    
    span {
        font-size: 1em; // 14px
    }
    
    strong {
       font-size: _em(20, 14); // 1.42857em === 20px 
    }
}
```

Please not that for this to work correctly, you need to use `62.5%`
as your root font-size. See `basic` module details about this.

### `_easing($key[, $value])`

- `$key` Key of the easing
- `$value` _(optional)_ Easing value

Shorthand function to get/set easing related values.

```scss
// Usage

.has-transition {
    transition: width 1s _easing('ease-in-out-expo');
}

.has-animation {
    animation-timing: _easing('ease-in-out-expo');
}
```

## Mixins

### `_triangle($size, $color, $direction)`

Creates a CSS triangle. Based on Zurb Foundation's solution.

- `$size` Size of triangle (px, em, etc.)
- `$color` Hmm... Color of the triangle I guess. Use a color directly, or a scheme key.
- `$direction` `top`, `right`, `bottom` or `left`

```scss
.tooltip {
    &::before {
        @include _triangle(
            12px,
            'tooltip.bg',
            bottom
        );
    }
}
```

### `_scrollbar([$overrides])`

Styles scrollbars where it's possible through CSS.
Supports IE10+ and Chrome. Rest of the browsers doesn't allow,
but hey, it's still a better solution than the shitty
JS scrollbars!

- `$overrides` _(optional)_ Override default values (coming from Scheme)

```scss
// Usage
body {
    @include _scrollbar;
}

// For just one div
.div {
    @include _scrollbar;
}

// Override default config
$_: _scheme('scrollbar', (
    'thumb-background': 'grey.main'
));

.div {
    @include _scrollbar;
}

// Direct value overwrite
.div {
    @include _scrollbar((
        'thumb-background': 'grey.main'
    ));
}

```

### `_on($breakpoint[, $up-down, $orientation])`

Handles responsive rules

- `$breakpoint` Breakpoint name defined in `_screen.scss`, or `from` value, or `retina`.
- `$up-down` _(optional)_ The given range and up/down, or `to` value.
- `$orientation` _(optional)_ `portrait`, `landscape`

```scss
// Usages
.title {
    font-size: 1.3em;
    
    // 0-700px
    @include _on(mobile) {
        font-size: 1em;
    }
    
    // 1024px and below
    @include _on(tablet, down) {
        font-size: 1em;
    }
    
    // 701px and up
    @include _on(tablet, up) {
        font-size: 1em;
    }
    
    // 1025px-1440px and portrait
    @include _on(small, false, portrait) {
        font-size: 1em;
    }
    
    // Only landscape
    @include _on(false, false, landscape) {
        font-size: 1em;
    }
    
    // Only on hdpi devices
    @include _on(retina) {
        font-size: 1em;
    }
}
```

## Modules

Modules are _optional_. Modules have output!
It's a good practice to load these modules at boot level.

```scss
// Common usage and flow
@import '~phrame/core';
@import 'myConfigOverrides';
@import '~phrame/module/reset';
@import '~phrame/module/basic';
@import '~phrame/module/expose';
@import 'myOwnStuff';
```

### `basic`

Includes some basic styles for your page.
Here is the full code of the module, we'll walk through it now.

```scss
html {
	// Root font size. 62.5% === 1em === 10px
    font-size: _typo('font-size.root');
    // We want the same size everywhere
    -webkit-text-size-adjust: 100%;
}

// Base font
body {
    font-family: _typo('font-family.sans');
    font-weight: normal;
    font-style: normal;
    line-height: _typo('line-height.root');
}

// Bbox sizing as it should be
*, *:after, *:before {
    box-sizing: border-box;
}
```

### `reset`

A simple reset. You might not need this in case you already have
your site built, or using some sort of framework, usually all
framework are shipped with reset styles.

## `expose`
This feature was usefull many times. It exposes your screen configuration (breakpoints)
as JSON in your CSS on `body::before`. This will let you to have
the same rules in JavaScript also, the rules are have to be maintained
only from one place.

```js
// Normalizes JSON
let normalize = s => s.replace(/^['"]+|\s+|\\|(;\s?})+|['"]$/g, '')

let breakpoints = JSON.parse(
    normalize(
        getComputedStyle(document.body, ':before').getPropertyValue('content')
    )
)
```

# Final words
As you can see, **Phrame** is not a full featured framework.
It's a higher order helper instead to kick start your own framework,
it helps to build your own system for your own custom needs.