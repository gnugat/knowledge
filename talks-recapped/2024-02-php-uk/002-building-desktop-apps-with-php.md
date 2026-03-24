# Building Desktop Apps with PHP

> **Talk Abstract**
>
> Have you ever wanted to write a desktop application? Using existing tools?
> Even with PHP? Well this talk is for you! Let me show you NativePHP - a
> Laravel Package that allows you to create desktop applications with PHP code.
> You can make use of native system APIs, clipboard access and more!

Speaker: Marcel Pociot
* [Video](https://www.youtube.com/watch?v=szi9OIef2oU)

## Recap

Marcel Pociot is the CTO at Beyond Code, a three-person company that builds
tools for PHP developers.
One of their products is Tinkerwell, a GUI application for running PHP code in
the context of a local application, similar to `php artisan tinker` but with a
proper interface.
The first version was written in Swift, which meant it was Mac-only.
Users on Windows and Linux asked for support, and rewriting the app natively
for each platform was not viable for a team of three.

### From Electron to PHP

Marcel looked at the two main options for cross-platform desktop apps.
Tauri uses Rust for the backend and JavaScript for the frontend.
Electron uses JavaScript throughout.
He knew JavaScript but not Rust, so the choice was clear: Electron.

An Electron application spawns two processes.
The main process (sometimes called the background process) is written in
JavaScript and has access to Node.js functionality: the filesystem, native UI
elements like the dock and menu bar, and operating system APIs like Touch ID.
The main process spawns a second process called the renderer, which is
essentially a Chromium browser rendering HTML and JavaScript.
Communication between the two happens over inter-process communication (IPC),
which Electron makes somewhat cumbersome.

Introducing PHP into this is straightforward.
The main process spawns `php artisan serve` alongside the renderer.
The renderer is pointed at the local URL instead of a static HTML file.
Communication from PHP back to the main process happens through a lightweight
HTTP API.
This gives PHP access to all the native capabilities the main process exposes.

The remaining problem is distribution.
Shipping a PHP application to end users means asking them to have the right
PHP version installed, along with any required extensions and configuration.
The solution is to bundle a statically compiled PHP binary with the application.
It is a single file with everything included, around 21 megabytes, the same
approach used by FrankenPHP and Laravel Herd.
This gives full control over the PHP version, extensions, and settings, with
no assumptions about what the user has installed.

This idea is what became NativePHP, a Laravel package co-created by Marcel and
Simon Hamp.

### Getting started

NativePHP is a Laravel package installed via Composer.
At the time of the talk it ships with an Electron driver (`nativephp/electron`).
A Tauri driver is in development, which will skip the local HTTP server entirely
and communicate with PHP directly via the C library.

After `composer require nativephp/electron`, running `php artisan native:install`
downloads the npm dependencies and Electron, and publishes a configuration file
and a service provider.
From there, `php artisan native:serve` starts the application in development
mode with hot reloading and developer tools available.

### The NativeAppServiceProvider

The service provider is the entry point for configuring the application.
Its `boot` method is called once the native layer is ready.
Here the developer defines how the application starts: window dimensions, title,
minimum size, whether the window should always be on top, which route to display,
and more.
The API is fluent:

```php
Window::open()
    ->width(250)
    ->height(400)
    ->minWidth(200)
    ->minHeight(300)
    ->alwaysOnTop()
    ->title('Hello London')
    ->route('dialogue')
    ->rememberState();
```

`rememberState()` persists the window's position and size between restarts,
which is the behavior users expect from a desktop application.

### Menu bar applications

NativePHP supports applications that live entirely in the menu bar, like Tuple
or CleanShot, with no dock icon.
Calling `MenuBar::create()` instead of `Window::open()` creates a menu bar app.
The same fluent API applies for width, height, and route.
If the application needs both a dock window and a menu bar icon, adding
`showDockIcon()` keeps the dock entry visible alongside the menu bar icon.

### Menus and events

The application menu can be fully customized.
NativePHP provides helpers for standard submenus (Edit, View, Window) that can
be chained alongside custom entries.
A custom menu item of type `event` dispatches a plain Laravel event when clicked:

```php
Menu::new()
    ->submenu('PHP UK', Menu::new()
        ->event(SettingsClickEvent::class, 'Preferences', hotkey: 'Cmd+,')
        ->separator()
        ->quit()
    )
    ->register();
```

Listening for this event is standard Laravel.
In the event listener, `Window::open()->id('settings')->route('settings')` opens
a new window.
The ID is important: if a window with that ID is already open, NativePHP brings
it to focus rather than opening a duplicate.

### Settings and real-time cross-window communication

NativePHP ships a `Settings` facade that provides a simple key-value store for
persisting application state.
In the demo, a settings window contains a dropdown for picking a background
color.
When the component mounts, it reads the stored color.
When the color changes, it writes the new value back.

The interesting part is propagating that change to another open window in real
time.
In a web application this would require WebSockets.
NativePHP offers a simpler path: Livewire supports a `native` modifier on event
listeners.
When a value in the `Settings` store changes, NativePHP dispatches a
`SettingChanged` event through the Electron layer.
A Livewire component in another window can listen for this event using the
`native` keyword and receive the update immediately, without polling, without
WebSockets, and without writing any JavaScript.

### Screenshot tool and facades

The demo builds a window picker: a full-screen transparent overlay that
highlights whatever window is under the cursor, then takes a screenshot of the
selected window.

NativePHP's `Screen` facade returns the list of connected displays, the current
cursor position, and the currently active windows.
A transparent, always-on-top, frameless, non-movable window is opened at
display dimensions.
As the mouse moves, the component queries the active windows (cached briefly to
avoid hammering the system on every mouse event), finds the topmost one whose
bounds contain the cursor, and positions a colored overlay div at the same
coordinates.
Clicking calls `screencapture` with the window ID, stores the result in the
clipboard, and closes the picker window.

The `Clipboard` facade exposes the current clipboard contents as text, HTML, or
image.
Combined with Livewire's `wire:poll`, the main window refreshes automatically
after a screenshot is taken and displays the captured image.

As a final flourish, Marcel prints the screenshot to a Bluetooth thermal printer
(a "cat printer") by shelling out to a Python script.
Output from the script that would normally go to Laravel's log file is
redirected to the developer tools console by NativePHP, which makes debugging
native integrations straightforward.

### What else NativePHP supports

The talk covers only a slice of the feature set.
At the time of the talk, NativePHP also supports:

- File and folder picker dialogues
- Touch ID and other OS authentication APIs
- SQLite via Eloquent with automatic migrations on startup
- Laravel queues with a background worker
- Laravel scheduler running on a per-second interval if needed
- Auto-updater with support for GitHub, S3, and DigitalOcean Spaces
- Global keyboard shortcuts that dispatch Laravel events
- System notifications
- Dock icon progress bars
- Context menus
- Deep linking from other applications
- Events for window lifecycle (minimize, maximize, close, move, mouse position)
