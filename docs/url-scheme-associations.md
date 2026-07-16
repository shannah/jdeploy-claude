# Custom URL Scheme Associations (and Singleton Mode)

jDeploy can register a native app as the handler for a **custom URL scheme**, so
that clicking a link like `assist:planner@foo?bar=buzz` (in a browser, an email, a
terminal, another app) launches — or wakes — your installed app and hands it the
URL. This guide covers the two `package.json` settings involved (`urlSchemes` and
`singleton`), how the URL reaches your app at runtime, and the concrete convention
the Agentic App Framework's **Assistor** app uses to turn such a URL into an agent
session.

## 1. Declaring the scheme — `jdeploy.urlSchemes`

Add a `urlSchemes` array under `jdeploy` in `package.json`. Each entry is a scheme
name (the part before the `:` in a URL), lowercase, without the `:` or `//`:

```json
{
  "name": "assistor",
  "jdeploy": {
    "jar": "target/assistor.jar",
    "javaVersion": "21",
    "title": "Assistor",
    "singleton": true,
    "urlSchemes": ["assist"]
  }
}
```

When the app is installed via a jDeploy native installer, the installer registers
the scheme with the OS:

- **macOS** — a `CFBundleURLTypes` entry in the app bundle's `Info.plist`.
- **Windows** — a `HKEY_CLASSES_ROOT\<scheme>` key with a `shell\open\command`
  pointing at the launcher.
- **Linux** — an `x-scheme-handler/<scheme>` MIME association in the `.desktop`
  file.

After installation, `assist:...` links resolve to your app on all three platforms.

> **Scheme naming.** Use a short, lowercase, app-specific scheme (`assist`,
> `myapp`, `acme-notes`). Avoid reserved/standard schemes (`http`, `mailto`,
> `file`, `tel`) and avoid collisions with other installed apps — the OS lets only
> one handler win per scheme.

## 2. Singleton mode — `jdeploy.singleton`

A URL scheme is far more useful when the app is a **single instance**. Set:

```json
"jdeploy": { "singleton": true }
```

With `singleton: true`, launching the app while it is already running does **not**
start a second process. Instead jDeploy routes the launch — including any URL that
triggered it — to the already-running instance, and brings its window to the front.
Without singleton mode, every `assist:` link would spawn a brand-new process (and, on
a cold link-click, you would still get one — but repeated clicks would pile up
windows).

The two settings are complementary:

| Setting | Effect |
|---|---|
| `urlSchemes` | Registers the scheme with the OS so links reach the app. |
| `singleton` | A running instance is focused and handed the URL, instead of a new process starting. |

Declare **both** for a clean "click a link → the running app handles it" experience.

## 3. How the URL reaches your app at runtime

jDeploy delivers scheme (and file) opens through the
`ca.weblite:jdeploy-desktop-lib-swing` library, which your app registers a handler
with:

```java
import ca.weblite.jdeploy.app.JDeployOpenHandler;
import ca.weblite.jdeploy.app.swing.JDeploySwingApp;

JDeploySwingApp.setOpenHandler(new JDeployOpenHandler() {
    @Override public void openURIs(java.util.List<java.net.URI> uris) {
        // Your app was opened with one or more scheme URLs.
    }
    @Override public void openFiles(java.util.List<java.io.File> files) { /* file assoc */ }
    @Override public void appActivated() { /* focus the window (singleton) */ }
});
```

Key runtime facts:

- **Cold start and warm start both arrive here.** If the app was not running, the
  library queues the URL and delivers it to `openURIs` as soon as the handler is
  registered — so you do **not** need to parse `argv` for a first-launch URL. If the
  app was already running (singleton), the second launch's URL is delivered to the
  first instance's `openURIs`.
- **Callbacks run on the Swing EDT.** Marshal off the EDT before doing slow work.
- Register the handler **after** your main window is visible, so `appActivated` has a
  window to focus.

## 4. Convention: `assist:<agent>@<host>?<query>` → an agent session

The Agentic App Framework builds on this seam. When an Assistor-style app receives a
scheme URL, the framework parses it as:

```
<scheme> : <agentName> @ <host> [ ? <query> ]
```

For `assist:planner@foo?bar=buzz`:

| Part | Value | Meaning |
|---|---|---|
| scheme | `assist` | The registered scheme (any scheme is accepted at runtime). |
| **agentName** | `planner` | The token between the `:` and the first `@` — **selects the agent**. |
| host | `foo` | Everything after `@` up to an optional `?`. |
| query | `bar=buzz` | Anything after `?`. |

Both the opaque form (`assist:planner@foo?bar=buzz`) and the authority form
(`assist://planner@foo?bar=buzz`) parse identically.

### What the framework does with it

1. **Resolves the agent by name** (case-insensitive) against the app's standalone
   agent definitions — the agents you manage in the **Agents** menu.
2. **Opens a fresh chat session** driven by that agent (reusing the current session
   if it is empty), titled `@<agentName>`.
3. **Seeds the session context with the agent's instructions and the full URL**, so
   the agent immediately has both its persona and the incoming URL to act on. The
   opening turn looks like:

   ```
   <the agent's instructions>

   Launched from URL: assist:planner@foo?bar=buzz
   ```

The session then behaves like any other — it uses the app's shared provider and
sign-in, streams responses, and persists.

### Failure handling (loud, non-fatal)

- **No agent with that name** → a chat notice: *"No agent named 'planner' — add it in
  the Agents menu."* No session is opened. Create the agent (matching the URL's name)
  and click the link again.
- **Unparseable URL** (no `@`, empty agent name, no scheme) → a chat notice
  *"Ignored URL (…): …"*. Other URLs in the same batch still process.

Neither case crashes the app; the window keeps running.

### Trying it out

1. Install the app from a jDeploy native installer (the scheme is only registered by
   the installer, not by `mvn`/`gradle` runs).
2. In the app's **Agents** menu, create an agent named exactly `planner` with some
   instructions (e.g. *"You are a planning assistant."*).
3. From a terminal or browser, open `assist:planner@foo?bar=buzz`:
   - macOS: `open "assist:planner@foo?bar=buzz"`
   - Linux: `xdg-open "assist:planner@foo?bar=buzz"`
   - Windows: `start "" "assist:planner@foo?bar=buzz"`
4. The running app focuses, opens a `@planner` session, and the agent starts with the
   URL in context.

## 5. Checklist

- [ ] `jdeploy.urlSchemes` lists your scheme (lowercase, no `:` or `//`).
- [ ] `jdeploy.singleton` is `true` so links wake the running instance.
- [ ] The app registers a `JDeployOpenHandler` after its window is shown (the
      framework does this for you).
- [ ] For the agent-launch convention, an agent whose **name** matches the URL exists
      before the link is clicked.
- [ ] Test with an **installed** build — schemes are registered by the installer, not
      by a dev-mode run.
