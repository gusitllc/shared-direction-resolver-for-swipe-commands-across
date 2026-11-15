# Shared direction resolver for swipe commands across object types and hosts, with scroll-edge admission of vertical swipes

**Defensive Publication** - published to establish prior art.
**Publisher:** Gus IT LLC.

## Abstract

A client application uses a single, rendering-independent gesture function to turn a released drag into one of four directions or no command, using a horizontal distance threshold, a vertical distance threshold and a dominance test. The same function serves a card deck for approval decisions, rendered both inside a mobile application shell and as a desktop web component, and a reader engine for heterogeneous object types such as messages, calendar events, notes, tasks and documents. Each object type is registered as a small adapter supplying list order, fetch, action replay, read marking, action labels and two secondary actions. In the reader, horizontal swipes move to the previous or next object, while vertical swipes are admitted only if the touch began with the content at the matching scroll edge or on the header, so scrolling never triggers a command. Objects open from a local cache and neighbours are prefetched.

## Technical Field

Touch user interfaces for mobile and web clients, specifically gesture recognition shared across content types and rendering hosts.

## Background

Applications that present many kinds of content, such as mail, calendar entries, notes, tasks, documents and approval requests, usually implement gestures separately for each screen. Thresholds and direction rules then drift between screens, users learn several grammars, and the same gesture means different things in different places. When an application has both a native shell and a desktop web front end, the gesture code is commonly written twice, and behaviour diverges further. A second problem is the collision between vertical swipe commands and vertical scrolling: on a long message or note, a downward drag is normally a scroll, so a vertical swipe command either fires accidentally while the user is reading or is not offered at all. Text entry fields add a third conflict, where a drag meant to select text is read as a command. Finally, navigating between objects by swipe feels slow if each object must be fetched from the network on arrival.

## Summary of the Disclosure

The disclosure describes a pure gesture module that has no dependency on the document model: it receives drag displacements and returns a direction or null, plus helper functions for card pose, card verdict, fly-out transform and keyboard mapping. Two hosts, a native-shell approval deck and a desktop web component, drive their rendering through one shared view module that consumes this gesture module, so the same input produces the same verdict on both. A reader engine registers each object type through an adapter and maps the four directions to previous, next, an ask box and a two-action sheet. A vertical command is admitted only when the scrollable body was at its top edge (for downward) or bottom edge (for upward) at touch start, or when the touch began on the header region. Drags that begin inside text entry fields are ignored. Objects open from a local cache first, and a configured number of neighbours on each side in the remembered list order is prefetched in the background.

## Detailed Description

The gesture module defines a frozen set of distances and a direction function. Given a released drag with horizontal displacement dx and vertical displacement dy, it first tests whether the drag is vertically dominant, meaning the absolute vertical displacement exceeds the absolute horizontal displacement. If vertically dominant and dy exceeds the vertical threshold, the direction is down; if dy is below the negative vertical threshold, the direction is up. Otherwise, if dx exceeds the horizontal threshold, the direction is right, and if dx is below its negative, left. Any other drag returns null. The module also provides a drag tracker with down, move and up methods that records the start point, reports a pose while moving (a vertical translation when vertically dominant, otherwise a translation plus a rotation proportional to dx and the opacity of approve and decline stamps), and on release returns a verdict computed from the direction. A separate tracker handles a feedback sheet with its own smaller vertical thresholds for send and cancel. A key map turns arrow keys into the same verdicts. None of these functions touch the document, so they can be exercised by replaying recorded gestures in tests against every host.

In the approval deck, the direction is mapped to verdicts: right approves, left declines, up opens a feedback sheet and down goes back. The deck's view module, which builds the card elements and attaches pointer handlers, imports the gesture module. A native application screen and a desktop web component both load this same view module; the web component loads the deck modules at runtime from a configured base path served by the mobile client, so the two hosts never hold separate copies.

The reader engine keeps a registry of adapters keyed by object kind. An adapter declares a noun and plural for messages to the user, a header selector, an optional direction-to-verb map, a list object whose order function returns the identifiers of the list the user came from, a fetch function for one object, a perform function used to replay a queued action, an optional function to mark the object seen, labels for actions, and the two actions offered behind a downward swipe. Registering an adapter also registers its fetch, perform and labels with a shared local cache. The default verb map is left to previous, right to next, up to an ask box and down to a two-action sheet.

To wire a screen, the engine sets touch handlers on the screen element. At touch start it records the start coordinates and the scroll state of the body: whether the scroll offset is at or within a pixel of the top, whether the offset plus visible height reaches the full content height within a pixel, and whether the touch target lies inside the header selector. If the touch starts inside an input, text area, select or editable region, nothing is recorded. At touch end it computes the displacement, resolves a direction through the gesture module, and applies the admission rule: a downward command requires the top edge or the header, an upward command requires the bottom edge or the header. Horizontal commands are always admitted. A configured list of kinds may opt out of the edge rule, for content such as photos. Touch cancel clears the start state. Handlers are assigned as properties so that a repaint re-wires without stacking listeners.

Moving to the previous or next object uses the list order; at either end the screen bounces toward that end, gives a light haptic tick and shows a short note that there are no more items. Every swipe also has a visible, labelled button for assistive technology, and the buttons for the two downward actions state the undo period.

Opening an object first checks the local cache unless a fresh read is requested. An account mark is taken before reading; if the account changed when the read completes, an error is thrown and nothing is kept or shown. If the device is online and actions from an earlier session are queued, a replay is started. On a cache hit the object is shown at once; otherwise the adapter fetches it and the result is stored. In both cases the engine prefetches, when online, the identifiers within a configured distance before and after the current one in list order, nearest first, skipping those already cached and discarding answers that arrive after an account change.

## Disclosed Features (each disclosed alone and in combination)

1. A system in which a single rendering-independent function maps the displacement of a released drag to one of right, left, up, down or none using a horizontal threshold, a vertical threshold and a vertical-dominance test.
2. The system wherein a native application screen and a desktop web component both drive their rendering through one shared view module that obtains drag poses and verdicts from that function, so the same gesture yields the same verdict on both hosts.
3. The system wherein each content type is registered as an adapter supplying list order, fetch, replay of a queued action, read marking, action labels and two secondary actions, and registration also registers fetch and replay with a shared local cache.
4. The method wherein in a reader horizontal directions move to the previous and next object in the remembered list order and vertical directions open an ask box and a two-action sheet.
5. The method wherein a downward command is admitted only if the touch began with the scrollable body at its top edge or on the header region, and an upward command only if the body was at its bottom edge or the touch began on the header region.
6. The method wherein a drag that begins inside a text entry field is never treated as a command.
7. The method wherein a configured set of content types may be exempted from the scroll-edge admission rule.
8. The method wherein an object opens from a local cache first and a configured number of neighbouring objects on each side in list order are prefetched in the background, with results discarded if the signed-in account changed.
9. The system wherein every swipe command also has a visible labelled button, and reaching either end of the list produces a bounce and a short notice instead of a navigation.

## Variations

- The direction function may use velocity at release in addition to displacement, or a combination of both.
- Scroll-edge state may be sampled continuously during the drag rather than only at touch start.
- The header admission region may be any designated drag handle rather than the content header.
- The shared gesture module may be compiled to native code for a native host while remaining the same source of thresholds.
- Diagonal drags may be resolved to the nearest axis with an angular tolerance instead of a strict dominance test.
- The verb map per content type may be user-configurable, for example swapping the meaning of left and right.

## Note on omitted parameters

Distance thresholds, the edge tolerance, animation timings, the number of prefetched neighbours and notice durations are implementation choices left out.
