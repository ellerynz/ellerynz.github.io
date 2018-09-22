---
layout: post
title: Prettier imports with webpack aliases and eslint
category: Coding
tags: webpack
year: 2018
month: 7
day: 22
summary: Feel gross writing `import Foo from '../../../../shared/components/..'` everywhere? Use an alias to the shared directory.
---

Feel gross writing `import Foo from '../../../../shared/components/..'` everywhere? Use an alias to the shared directory. That way it is as easy as `Shared/components/...`.  eslint won't be able to pick up on the alias but we can fix that. Here's how we can do this in a rails/webpacker codebase.

### Add an alias to webpack:
A `webpack/custom.js` file makes sense to putting all your custom config to be used in all environments. The output of the custom.js is used in the environment.js (assuming webpacker). Now we can make 'Shared' is a shortcut to `app/javascript/shared`  that can be used anywhere in our javascript folder.

```
// webpack/custom.js
const path = require('path');

module.exports = {
  resolve: {
    alias: {
      Shared: path.resolve(__dirname, '..', '..', 'app', 'javascript', 'shared'),
    },
  },
};

// webpack/envronment.js
const { environment } = require('@rails/webpacker');
const customConfig = require('./custom');

environment.config.merge(customConfig);

module.exports = environment;
```

### Usage with eslint
eslint will be lighting up like a christmas tree—it's got no idea about this alias that we've set up so is spamming us with:

```
import/no-unresolved — Unable to resolve path to module 'Shared/components/...'.
import/extensions — Missing file extension for 'Shared/components/...'
```

The simple fix is adding the dev dependency `yarn add -D eslint-import-resolver-webpack` (assuming you already have `eslint-plugin-import` as a dev dependency too). This helps eslint know where to look for modules when using webpack.

We'll need to update our `eslintrc` to add the import-resolver settings. It'll try and guess where your webpack config is or you can specify where it is as a config option. We'll use the config option.

```
// eslintrc.json
"settings": {
  "import/resolver": {
    "webpack": {
      "config": "config/webpack/custom.js"
    }
  }
}
```

### Debugging setup

If the above doesn't get eslint to quieten down (and you're spending a long time googling to see where you went wrong...) the following line may help.:

`DEBUG=eslint-plugin-import:* yarn eslint path/to/your/file.js`

The result is some very verbose output but for me it pointed out a simple spelling mistake which got everything working once resolved.

### Final setup

Here's the final setup using the following folder structure:

```
app/javascript/shared/components/...
config/webpack/custom.js
config/webpack/environment.js
eslintrc.json
package.json
```

```
// package.json
"devDependencies": {
  "eslint-import-resolver-webpack": "^0.10.1",
  "eslint-plugin-import": "^2.10.0",
}

// eslintrc.json
"settings": {
  "import/resolver": {
    "webpack": {
      "config": "config/webpack/custom.js"
    }
  }
}

// webpack/custom.js
const path = require('path');

module.exports = {
  resolve: {
    alias: {
      Shared: path.resolve(__dirname, '..', '..', 'app', 'javascript', 'shared'),
    },
  },
};

// webpack/envronment.js
const { environment } = require('@rails/webpacker');
const customConfig = require('./custom');

environment.config.merge(customConfig);

module.exports = environment;
```

### Links

- https://webpack.js.org/configuration/resolve/#resolve
- https://github.com/benmosher/eslint-plugin-import
- https://www.npmjs.com/package/eslint-import-resolver-webpack