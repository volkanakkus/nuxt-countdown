<template>
  <div>
    Basic usage with Date object
    <Countdown
      v-slot="{ years, days, hours, minutes, seconds }"
      with-years
      :date="new Date('Oct 19, 2026 16:50:30')"
    >
      Time Remaining: {{ years }} years, {{ days }} days, {{ hours }} hours,
      {{ minutes }} minutes, {{ seconds }} seconds.
    </Countdown>

    <br />
    Basic usage with time in milliseconds
    <Countdown
      v-slot="{ days, hours, minutes, seconds }"
      :time="2 * 24 * 60 * 60 * 1000"
    >
      Time Remaining: {{ days }} days, {{ hours }} hours, {{ minutes }} minutes,
      {{ seconds }} seconds.
    </Countdown>

    <br />

    Custom interval
    <Countdown
      v-slot="{ days, hours, minutes, seconds, milliseconds }"
      :time="customTime"
      :interval="100"
    >
      New Year Countdown: {{ days }} days, {{ hours }} hours,
      {{ minutes }} minutes, {{ seconds }}.{{ Math.floor(milliseconds / 100) }}
      seconds.
    </Countdown>

    <br />

    Total time remaining
    <Countdown
      v-slot="{ totalDays, totalHours, totalMinutes, totalSeconds }"
      :date="new Date('Oct 19, 2026 16:50:30')"
    >
      <br />

      Time Remaining: {{ totalDays }} days or {{ totalHours }} hours or
      {{ totalMinutes }} minutes or {{ totalSeconds }} seconds.
    </Countdown>

    <br />

    Transform slot props
    <Countdown
      v-slot="{ days, hours, minutes, seconds }"
      :time="2 * 24 * 60 * 60 * 1000"
      :transform="transformSlotProps"
    >
      Time Remaining: {{ days }} days, {{ hours }} hours, {{ minutes }} minutes,
      {{ seconds }} seconds.
    </Countdown>

    <br />

    Countdown on demand
    <br />
    <button type="button" :disabled="counting" @click="startCountdown">
      <Countdown
        v-if="counting"
        v-slot="{ totalSeconds }"
        :time="60500"
        @end="onCountdownEnd"
      >
        Fetch again {{ totalSeconds }} seconds later
      </Countdown>
      <span v-else>Fetch Verification Code</span>
    </button>
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue";

// Custom interval
const now: Date = new Date();
const newYear: Date = new Date(now.getFullYear() + 1, 0, 1);
const customTime: Ref<number> = ref(newYear.getTime() - now.getTime());

// Transform slot props
const transformSlotProps = (props: Record<string, number>) => {
  const formattedProps: Record<string, string> = {};
  Object.entries(props).forEach(([key, value]) => {
    formattedProps[key] = (value as number) < 10 ? `0${value}` : String(value);
  });
  return formattedProps;
};

// Countdown on demand
const counting: Ref<boolean> = ref(false);
const startCountdown = () => {
  counting.value = true;
};
const onCountdownEnd = () => {
  counting.value = false;
};
</script>
