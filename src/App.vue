<script setup lang="ts">
import { computed, ref, watch } from "vue";
import EventList from "./components/EventList.vue";
import { initialEvents } from "./data/events";
import { seatsLeft } from "./format";
import type { CampusEvent } from "./types";

const STORAGE_KEY = "campus-events:my-bookings";

// A reactive copy of the hard-coded events
const events = ref<CampusEvent[]>(initialEvents.map(e => ({ ...e })));

function loadBookings(): number[] {
  try {
    const parsed: unknown = JSON.parse(localStorage.getItem(STORAGE_KEY) ?? "[]");
    return Array.isArray(parsed) ? parsed.filter((id): id is number => typeof id === "number") : [];
  } catch {
    return [];
  }
}

const myBookings = ref<number[]>(loadBookings());
for (const id of myBookings.value) {
  const e = events.value.find(e => e.id === id);
  if (e) e.booked++;
}

const bookingCount = computed(() => myBookings.value.length);

function onBook(id: number) {
  const e = events.value.find(e => e.id === id);
  if (!e || seatsLeft(e) === 0) return;
  e.booked++;
  myBookings.value.push(id);
}

watch(myBookings, ids => localStorage.setItem(STORAGE_KEY, JSON.stringify(ids)), { deep: true });
</script>

<template>
  <main>
    <header>
      <h1>Campus Events</h1>
      <p class="mine">You have booked {{ bookingCount }} {{ bookingCount === 1 ? "seat" : "seats" }}.</p>
    </header>
    <EventList :events="events" @book="onBook" />
  </main>
</template>
<style scoped>
header { margin-bottom: 16px; }
.mine { color: #6b7280; margin: 0; }
</style>
