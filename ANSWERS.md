# Week 3: Explain it

## 1. Using the array index as `:key`

With `:key="index"`, Vue identifies each card by its position, not by the event. When I filter (for example search "lab"), Git Workshop moves from index 1 to index 0, so Vue reuses the component that was at index 0 (Intro to Hackathons) and only changes its props. Any local state, like an open details panel, stays with the position, so it shows up on the wrong event. With `:key="e.id"` Vue keeps each card's component with its own event and removes the ones that are filtered out.

## 2. Why `visible` is a `computed`

A computed is cached: Vue remembers the result and only recalculates it when something it reads changes (`search`, `category` or `events`). A function called from the template runs again on every re-render, even when the re-render has nothing to do with the filter. When I type in the search box, `search` changes, so `visible` is recalculated once, and the count, the `v-if` and the `v-for` all use that same result. With a function, the filter would run three times per render, once for each place it is called.

## 3. Why EventCard emits `book` instead of changing `event.booked`

First, props are read-only in the child: `props.event.booked++` makes Vue show a warning, and it would change the parent's data without the parent knowing. Second, App owns the events, so it is the one that decides what booking means: it checks that a seat is left, increases `booked` and also adds the id to `myBookings`. This keeps EventCard simple and reusable, because it only reports "the user wants to book event 7".

## 4. `const { booked } = reactive(event); booked++` does not update the page

Destructuring reads `booked` once and copies the number into a normal local variable. Vue tracks changes through the reactive object (`event.booked`), not through this copy, so changing the copy does not change the object and nothing re-renders (and since it is a `const`, `booked++` would even throw an error). To make it work I have to change the property itself, `event.booked++`, or use `toRefs(event)` to keep the link.

## 5. Where `myBookings` lives

`myBookings` lives in `App.vue`, next to `events`. App owns the events and shows the booking count in the header, so it is the closest common parent of everything that uses this state. EventCard is one card per event and is destroyed when the filter hides it, so each card would only know its own bookings and lose them. EventList only filters and displays the list, and it does not show the header count.
