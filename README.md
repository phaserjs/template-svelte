# Phaser Svelte Template

This is a Phaser 3 project template that uses the Svelte framework, TypeScript and Vite for bundling. It includes a bridge for Svelte to Phaser game communication, hot-reloading for quick development workflow and scripts to generate production-ready builds.

### Versions

This template has been updated for:

- [Phaser 3.80.1](https://github.com/phaserjs/phaser)
- [Svelte 4.2.7](https://github.com/sveltejs/kit)
- [Vite 5.1.4](https://github.com/vitejs/vite)
- [TypeScript 5.3.3](https://github.com/microsoft/TypeScript)

![screenshot](screenshot.png)

## Requirements

[Node.js](https://nodejs.org) is required to install dependencies and run scripts via `npm`.

## Available Commands

| Command | Description |
|---------|-------------|
| `npm install` | Install project dependencies |
| `npm run dev` | Launch a development web server |
| `npm run build` | Create a production build in the `build` folder |

## Writing Code

After cloning the repo, run `npm install` from your project directory. Then, you can start the local development server by running `npm run dev`.

The local development server runs on `http://localhost:8080` by default. Please see the Vite documentation if you wish to change this, or add SSL support.

Once the server is running you can edit any of the files in the `src` folder. Vite will automatically recompile your code and then reload the browser.

## Template Project Structure

We have provided a default project structure to get you started. This is as follows:

- `src` - Contains the Svelte source code.
- `src/app.html` - The html Svelte container.
- `src/app.d.ts` - Global TypeScript declarations, provide types information.
- `src/routes/+layout.svelte` - Svelte layout component. Here, the page title and the global styles are defined.
- `src/+page.svelte` - Svelte page that integrates the functionality of the game created with Phaser.
- `src/game` - Contains the game source code.
- `src/game/PhaserGame.svelte` - The Svelte component that initializes the Phaser Game and serve like a bridge between Svelte and Phaser.

- `src/game/EventBus.ts` - A simple event bus to communicate between Svelte and Phaser.
- `src/game/main.ts` - The main **game** entry point. This contains the game configuration and start the game.
- `src/game/scenes/` - The Phaser Scenes are in this folder.
- `static/assets` - Contains the static assets used by the game.

## Svelte Bridge

The `PhaserGame.svelte` component is the bridge between Svelte and Phaser. It initializes the Phaser game and passes events between the two.

To communicate between Svelte and Phaser, you can use the **EventBus.ts** file. This is a simple event bus that allows you to emit and listen for events from both Svelte and Phaser.

```js
// In Svelte
import { EventBus } from './EventBus';

// Emit an event
EventBus.emit('event-name', data);

// In Phaser
// Listen for an event
EventBus.on('event-name', (data) => {
    // Do something with the data
});
```

In addition to this, the `PhaserGame` component exposes the Phaser game instance along with the most recently active Phaser Scene. You can pick these up from Svelte via `phaserRef prop`.

Once exposed, you can access them like any regular reference.

## Phaser Scene Handling

In Phaser, the Scene is the lifeblood of your game. It is where you sprites, game logic and all of the Phaser systems live. You can also have multiple scenes running at the same time. This template provides a way to obtain the current active scene from Svelte.

You can get the current Phaser Scene from the component event `"current-active-scene"`. In order to do this, you need to emit the event `"current-scene-ready"` from the Phaser Scene class. This event should be emitted when the scene is ready to be used. You can see this done in all of the Scenes in our template.

**Important**: When you add a new Scene to your game, make sure you expose to Svelte by emitting the `"current-scene-ready"` event via the `EventBus`, like this:


```js
class MyScene extends Phaser.Scene
{
    constructor ()
    {
        super('MyScene');
    }

    create ()
    {
        // Your Game Objects and logic here

        // At the end of create method:
        EventBus.emit('current-scene-ready', this);
    }
}
```

You don't have to emit this event if you don't need to access the specific scene from Svelte. Also, you don't have to emit it at the end of `create`, you can emit it at any point. For example, should your Scene be waiting for a network request or API call to complete, it could emit the event once that data is ready.

### Svelte Component Example

Here's an example of how to access Phaser data for use in a Svelte Component:

```js
// In a parent component
<script lang="ts">
    import type { Scene } from "phaser";
    import PhaserGame, { type TPhaserRef } from "game/PhaserGame.svelte"; // We provide the type TPhaserRef but this route is an example. You should use the correct path to the PhaserGame component.

    let phaserRef: TPhaserRef = { game: null, scene: null};

    const onCurrentActiveScene = (scene) => {
        
        // This is invoked

    }

</script>

<PhaserGame phaserRef={phaserRef} currentActiveScene={onCurrentActiveScene} />
```

In the code above, you can get a reference to the current Phaser Game instance and the current Scene by creating a reference with a variable `let phaserRef` and assign to PhaserGame component.

From this reference, the game instance is available via `phaserRef.game` and the most recently active Scene via `phaserRef.scene`.

The `onCurrentActiveScene` callback will also be invoked whenever the the Phaser Scene changes, as long as you emit the event via the EventBus, as outlined above.

## Handling Assets

To load your static games files such as audio files, images, videos, etc place them into the `static/assets` folder. Then you can use this path in the Loader calls within Phaser:

```js
preload ()
{
    //  This is an example of loading a static image
    //  from the static/assets folder:
    this.load.image('background', 'assets/bg.png');
}
```

When you issue the `npm run build` command, all static assets are automatically copied to the `build/assets` folder.

## Deploying to Production

After you run the `npm run build` command, your code will be built into a single bundle and saved to the `build` folder, along with any other assets your project imported, or stored in the public assets folder.

In order to deploy your game, you will need to upload *all* of the contents of the `build` folder to a public facing web server.

## Customizing the Template

### Vite

If you want to customize your build, such as adding plugin (i.e. for loading CSS or fonts), you can modify the `vite/config.*.mjs` file for cross-project changes, or you can modify and/or create new configuration files and target them in specific npm tasks inside of `package.json`. Please see the [Vite documentation](https://vitejs.dev/) for more information.

## Warning

Normally, SvelteKit renders your page on the server first and sends that HTML to the client where it's hydrated. If you set ssr to false, it renders an empty 'shell' page instead. This is useful if your page is unable to be rendered on the server (because you use browser-only globals like document for example).

Phaser needs to run on the client, therefore in the file `src/routes/+layout.js` we have added the line:
```javascript
export const ssr = false;
```
Please do not modify this line unless you know what you are doing and can resolve all related issues with SSR.

## Join the Phaser Community!

We love to see what developers like you create with Phaser! It really motivates us to keep improving. So please join our community and show-off your work 😄

**Visit:** The [Phaser website](https://phaser.io) and follow on [Phaser Twitter](https://twitter.com/phaser_)<br />
**Play:** Some of the amazing games [#madewithphaser](https://twitter.com/search?q=%23madewithphaser&src=typed_query&f=live)<br />
**Learn:** [API Docs](https://newdocs.phaser.io), [Support Forum](https://phaser.discourse.group/) and [StackOverflow](https://stackoverflow.com/questions/tagged/phaser-framework)<br />
**Discord:** Join us on [Discord](https://discord.gg/phaser)<br />
**Code:** 2000+ [Examples](https://labs.phaser.io)<br />
**Read:** The [Phaser World](https://phaser.io/community/newsletter) Newsletter<br />

Created by [Phaser Studio](mailto:support@phaser.io). Powered by coffee, anime, pixels and love.

The Phaser logo and characters are &copy; 2011 - 2024 Phaser Studio Inc.

All rights reserved.
