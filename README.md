# @alexandernygard/drive

Open, create and share documents in [Drive](https://drive.nyggis.com) from the office apps (word2, xl2,
ppt2), with the same share dialog, avatars and file icons as the dashboard.

| Import | What |
|---|---|
| `@alexandernygard/drive` | `getDrive()` (the API client), `fileFromUrl()`, types, `APPS`, `template()`, formatting helpers |
| `@alexandernygard/drive/core` | The same without the browser client, for Workers |
| `@alexandernygard/drive/react` | `ShareDialog`, `DriveFileButton`, `AvatarStack`, `PersonAvatar`, `KindIcon`, `DriveLogo`, `useDriveUser` |

## Opening a document in an app

The dashboard opens documents as `https://<app>/?file=<id>`. Connect to the drive's room with a ticket
instead of the app's own Worker:

```ts
import { CollabProvider } from "@alexandernygard/collab"
import { deniedMessage, fileFromUrl, getDrive } from "@alexandernygard/drive"

const drive = getDrive()           // production origins, or localhost ports in dev
const id = fileFromUrl()
let provider!: CollabProvider
provider = new CollabProvider(
  drive.roomOptions(
    id,
    (t) => setItem(t.item),         // every reconnect: current name, sharing, t.readOnly
    (e) => {                        // signed out / no access / deleted: retrying won't help
      showBanner(deniedMessage(e), e.status === 401 && drive.loginUrl())
      setTimeout(() => provider.disconnect())
    },
  ),
)
```

The room already holds the document (the drive seeds new ones), so don't seed it from the app. Use the
person's account for presence: `drive.me()` gives `{ id, name, email, picture }`.

In the header, `<DriveFileButton fileId={id} item={item} peers={peers} />` shows a link back to the file's
folder, who else is here, and Share (the drive's dialog, in place). For local files, offer
`drive.create({ kind, name, content })` as "Save to Drive" and switch to the returned id.

## Setup in an app

- Dedupe React and the component library in Vite, so the dialog uses the app's copies:
  `resolve: { dedupe: ["yjs", "lib0", "y-protocols", "react", "react-dom", "@alexandernygard/components", "lucide-react"] }`.
- Let Tailwind see the components' classes, after the CSS imports:
  `@source "../node_modules/@alexandernygard/drive/dist";`
- Add the app's origin to the drive's `APP_ORIGINS`, or its requests are refused.

The Drive service itself (dashboard, API and document rooms) lives in
[alexandernygard/drive](https://github.com/alexandernygard/drive) and uses this package too: the Worker
imports `@alexandernygard/drive/core` for the shared types and templates. Keep the API backwards
compatible: the apps update this package on their own schedule, so add fields and routes rather than
renaming or removing them.

## Developing

```bash
pnpm install
pnpm test
pnpm build
```

To try a change in an app before releasing, run `pnpm pack` here and `pnpm add <path to the .tgz>` in the
app. Don't commit that dependency.

Release with Changesets: run `pnpm changeset` with each change (pick the bump, one-sentence summary) and
commit the file. To release, `pnpm version-packages`, commit and push, then `pnpm release` and
`git push --tags`.
