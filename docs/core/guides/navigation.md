---
title: Navigation
---

# Navigation

## Tracking the reading position

`onLocationChange` fires with a `Locator` whenever the position changes, for example when the user
turns a page. Save it to reopen the book where the user left off, through `file.initialLocation`:

```tsx
const [location, setLocation] = useState<Locator>();

<ReadiumView
  file={{ url: path, initialLocation: savedLocation }}
  onLocationChange={(locator) => {
    setLocation(locator);
    save(bookId, locator);
  }}
/>;
```

## Moving the reader

The view's ref moves the reader:

```tsx
import { useRef } from 'react';
import { ReadiumView, type ReadiumViewRef } from 'react-native-readium';

const ref = useRef<ReadiumViewRef>(null);

<>
  <ReadiumView ref={ref} file={file} />
  <Button title="Previous" onPress={() => ref.current?.goBackward()} />
  <Button title="Next" onPress={() => ref.current?.goForward()} />
</>;

// Jump to a chapter from the table of contents, a bookmark or a search result.
ref.current?.goTo(locator);
```

| Method          |                                                           |
| --------------- | --------------------------------------------------------- |
| `goTo(locator)` | Go to a location: a chapter, a bookmark, a search result. |
| `goForward()`   | The next page.                                            |
| `goBackward()`  | The previous page.                                        |

To open a table-of-contents entry, build a `Locator` from its `Link`:

```tsx
ref.current?.goTo({
  href: link.href,
  type: link.type || 'application/xhtml+xml',
  title: link.title || '',
  locations: { progression: 0 },
});
```

The `positions` from [`onPublicationReady`](./opening-publications.md#when-the-publication-is-ready)
are ready-made locators, one per position, for a page slider or "go to page".

## Drawing your own chrome

An app that draws its own bars and page label can take over three things:

```tsx
<ReadiumView
  file={file}
  hidePageNumbers
  onTap={({ point }) => setChromeVisible((visible) => !visible)}
/>
```

`onTap` fires for a tap on the page that nothing else used. A tap on a link, a selection, a
decoration or a page-turn edge never reaches it. `point` is where the tap landed, in points on iOS
and in pixels on Android.

On iOS, a view with no `onTap` toggles its own navigation bar on such a tap. With `onTap` set, it
no longer does, and it keeps the bar and the VoiceOver previous and next toolbar hidden, including
while VoiceOver runs. The app's own controls replace them.

`hidePageNumbers` hides the position label the reader draws over the page ("12 / 340"). It is off
by default.

iOS turns pages on a tap in the left or right edge of the page, as Android does. The top and
bottom edges no longer turn pages.

On Android, a PDF has no tap listener yet, so `onTap` never fires there. With a scrolling layout
nothing turns pages by tap, so every tap on the page reaches `onTap`.

## Zoom (PDF)

PDFs can be magnified through the ref: `zoomIn()`, `zoomOut()`, `setZoom(scale)` and
`resetZoom()`. On web, `onZoomChange` reports the scale, including pinches; on iOS and Android
the platform PDF viewers zoom on a pinch by themselves and report nothing back.
