# `@soundworks/plugin-platform`

> [`soundworks`](https://github.com/collective-soundworks/soundworks) plugin that checks the availability of features required by the application, and their initialization. It also provides an entry point to the features that may require a user gesture (e.g. clicking on the screen to resume an audio context). The plugin can also be used to simply add a splash screen to the application.

## Table of Contents

<!-- toc -->

- [Installation](#installation)
- [Example](#example)
- [Usage](#usage)
  * [Server installation](#server-installation)
    + [Registering the plugin](#registering-the-plugin)
    + [Requiring the plugin](#requiring-the-plugin)
  * [Client installation](#client-installation)
    + [Registering the plugin](#registering-the-plugin-1)
    + [Requiring the plugin](#requiring-the-plugin-1)
  * [Adding new features](#adding-new-features)
    + [Lifecycle](#lifecycle)
    + [Example - `@ircam/devicemotion`](#example---ircamdevicemotion)
- [Credits](#credits)
- [License](#license)

<!-- tocstop -->

## Installation

```sh
npm install @soundworks/plugin-platform --save
```

## Example

A working example can be found in the [example](https://github.com/collective-soundworks/soundworks-examples) repository.

## Usage

### Server installation

#### Registering the plugin

```js
// index.js
import { Server } from '@soundworks/core/server';
import pluginPlatformFactory from '@soundworks/plugin-platform/server';

const server = new Server();
server.pluginManager.register('platform', pluginPlatformFactory, {}, []);
```

#### Requiring the plugin

```js
// MyExperience.js
import { AbstractExperience } from '@soundworks/core/server';

class MyExperience extends AbstractExperience {
  constructor(server, clientType) {
    super(server, clientType);
    // require plugin in the experience
    this.platform = this.require('platform');
  }
}
```

### Client installation

#### Registering the plugin

```js
// index.js
import { Client } from '@soundworks/core/client';
import pluginPlatformFactory from '@soundworks/plugin-platform/client';

const client = new Client();
const AudioContext = window.AudioContext || window.webkitAudioContext;
const audioContext = new AudioContext();

client.pluginManager.register('platform', pluginPlatformFactory, {
  features: [
    ['web-audio', audioContext],
  ]
}, []);
```

#### Requiring the plugin

```js
// MyExperience.js
import { Experience } from '@soundworks/core/client';

class MyExperience extends Experience {
  constructor(client) {
    super(client);
    // require plugin in the experience
    this.platform = this.require('platform');
  }
}
```

### Adding new features

By default, the plugin only ships the feature dedicated to resuming a given `audioContext`, however user-defined features can be added for specific use cases (devicemotion permission, etc., see the [adding new features](#adding-new-features) section for more informations). This `audio-context` definition also contains additional checks for weird quirks found in the wild (broken `sampleRate` on iOS, etc.).

This plugin also tries to wakelock the device using the [nosleep.js](https://github.com/richtr/NoSleep.js/) library.

By default, the `soundworks-template` ships all the views to interact with the plugin.

#### Lifecycle

The initialization lifecycle of a feature follows these steps:

> available --> authorize --> initialize --> finalize

- The `available` and `authorize` steps are executed when the plugin starts.
- The `initialize` and `finalize` steps are executed when calling the `onUserGesture(e)` method, that should be called on a `mouseup` or `touchend` event (cf. https://developers.google.com/web/updates/2017/09/autoplay-policy-changes).

If all the steps of every feature resolves to `true` the plugin is marked as `ready`, else it passes an `errored` state and prevents the application from launching.

Each of these steps can be defined by a function that must return a `Promise` resolving on `true` if the lifecyle can continue or `false`. If the function is not provided, the lifecycle step is simply ignored:

```
- @param {Object} def - Definition of the feature.
  + @param {String} def.id
  + @param {Function : Promise.resolve(true|false)} [def.available=undefined]
  + @param {Function : Promise.resolve(true|false)} [def.authorize=undefined]
  + @param {Function : Promise.resolve(true|false)} [def.initialize=undefined]
  + @param {Function : Promise.resolve(true|false)} [def.finalize=undefined]
```

#### Example - `@ircam/devicemotion`

For example, requiring permission for motion sensors would lead to the following initialization of the plugin. The example uses the [@ircam/devicemotion](https://github.com/ircam-jstools/devicemotion) library, dedicated to providing a consistent interface and behavior across browsers.

```js
// index.js
import { Client } from '@soundworks/core/client';
import pluginPlatformFactory from '@soundworks/plugin-platform/client';
import devicemotion from '@ircam/devicemotion';

const client = new Client();
// register device motion feature
pluginPlatformFactory.addFeatureDefinition({
  id: 'devicemotion',
  initialize: async () => {
    const result = await devicemotion.requestPermission();
    return (result === 'granted' ? true : false);
  },
});

client.pluginManager.register('platform', pluginPlatformFactory, {
  features: [
    ['web-audio', audioContext],
    ['devicemotion'],
  ]
}, []);
```

## Credits

The code has been initiated in the framework of the WAVE and CoSiMa research projects, funded by the French National Research Agency (ANR).

## License

BSD-3-Clause
