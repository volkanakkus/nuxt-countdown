<template>
  <component :is="tag">
    <slot v-bind="transformedProps" />
  </component>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onBeforeUnmount, type Ref } from "vue";

const MILLISECONDS_SECOND = 1000;
const MILLISECONDS_MINUTE = 60 * MILLISECONDS_SECOND;
const MILLISECONDS_HOUR = 60 * MILLISECONDS_MINUTE;
const MILLISECONDS_DAY = 24 * MILLISECONDS_HOUR;
const MILLISECONDS_YEAR = 365 * MILLISECONDS_DAY;
const EVENT_ABORT = "abort";
const EVENT_END = "end";
const EVENT_PROGRESS = "progress";
const EVENT_START = "start";
const EVENT_VISIBILITY_CHANGE = "visibilitychange";

const props = defineProps({
  /**
   * Starts the countdown automatically when initialized.
   */
  autoStart: {
    type: Boolean,
    default: true,
  },
  /**
   * Emits the countdown events.
   */
  emitEvents: {
    type: Boolean,
    default: true,
  },
  /**
   * The interval time (in milliseconds) of the countdown progress.
   */
  interval: {
    type: Number,
    default: 1000,
    validator: (value: number) => value >= 0,
  },
  /**
   * Generate the current time of a specific time zone.
   */
  now: {
    type: Function,
    default: () => Date.now(),
  },
  /**
   * The tag name of the component's root element.
   */
  tag: {
    type: String,
    default: "div",
  },
  /**
   * The time (in milliseconds) to count down from.
   */
  time: {
    type: Number,
    default: 0,
    validator: (value: number) => value >= 0,
  },
  /**
   * The time (in Date) to count down from.
   * @type {Date}
   */
  date: {
    type: Date,
    default: null,
    validator: (value: Date) => value.getTime() >= 0,
  },
  /**
   * Transforms the output props before render.
   */
  transform: {
    type: Function,
    default: (props: unknown) => props,
  },
  /**
   * Is years included in the countdown.
   */
  withYears: {
    type: Boolean,
    default: false,
  },
});

/**Emits---------------- */
// TODO: Remove the eslint-disable-line after the issue is fixed.
// eslint-disable-next-line vue/valid-define-emits
const emit = defineEmits([EVENT_ABORT, EVENT_END, EVENT_PROGRESS, EVENT_START]);
/**--------------------- */

/**State---------------- */
// It is counting down.
const counting: Ref<boolean> = ref(false);
// The absolute end time.
const endTime: Ref<number> = ref(0);
// The remaining milliseconds.
const totalMilliseconds: Ref<number> = ref(0);
// The request ID of the requestAnimationFrame.
const requestId: Ref<number> = ref(0);
/**------------------- */

/**Computed---------------- */
/**
 * Time
 * @returns {number} The computed value.
 */
const time = computed(() => {
  return props.date?.getTime() - props.now() || props.time;
});
/**
 * Remaining years.
 * @returns {number} The computed value.
 */
const years = computed(() => {
  return props.withYears
    ? Math.floor(totalMilliseconds.value / MILLISECONDS_YEAR)
    : 0;
});
/**
 * Remaining days.
 * @returns {number} The computed value.
 */
const days = computed(() => {
  if (props.withYears) {
    return Math.floor(
      (totalMilliseconds.value % MILLISECONDS_YEAR) / MILLISECONDS_DAY
    );
  } else {
    return Math.floor(totalMilliseconds.value / MILLISECONDS_DAY);
  }
});
/**
 * Remaining hours.
 * @returns {number} The computed value.
 */
const hours = computed(() => {
  return Math.floor(
    (totalMilliseconds.value % MILLISECONDS_DAY) / MILLISECONDS_HOUR
  );
});
/**
 * Remaining minutes.
 * @returns {number} The computed value.
 * */
const minutes = computed(() => {
  return Math.floor(
    (totalMilliseconds.value % MILLISECONDS_HOUR) / MILLISECONDS_MINUTE
  );
});
/**
 * Remaining seconds.
 * @returns {number} The computed value.
 * */
const seconds = computed(() => {
  return Math.floor(
    (totalMilliseconds.value % MILLISECONDS_MINUTE) / MILLISECONDS_SECOND
  );
});
/**
 * Remaining milliseconds.
 * @returns {number} The computed value.
 * */
const milliseconds = computed(() => {
  return Math.floor(totalMilliseconds.value % MILLISECONDS_SECOND);
});
/**
 * Total remaining days.
 * @returns {number} The computed value.
 * */
const totalDays = computed(() => {
  return props.withYears ? days.value + years.value * 365 : days.value;
});
/**
 * Total remaining hours.
 * @returns {number} The computed value.
 * */
const totalHours = computed(() => {
  return Math.floor(totalMilliseconds.value / MILLISECONDS_HOUR);
});
/**
 * Total remaining minutes.
 * @returns {number} The computed value.
 * */
const totalMinutes = computed(() => {
  return Math.floor(totalMilliseconds.value / MILLISECONDS_MINUTE);
});
/**
 * Total remaining seconds.
 * @returns {number} The computed value.
 * */
const totalSeconds = computed(() => {
  return Math.floor(totalMilliseconds.value / MILLISECONDS_SECOND);
});
/**------------------- */

/**Lifecycles--------- */
onMounted(() => {
  totalMilliseconds.value = time.value;
  endTime.value = props.now() + time.value;

  if (props.autoStart) {
    start();
  }

  document.addEventListener(EVENT_VISIBILITY_CHANGE, handleVisibilityChange);
});

onBeforeUnmount(() => {
  document.removeEventListener(EVENT_VISIBILITY_CHANGE, handleVisibilityChange);
  pause();
});
/**------------------- */

/**Methods------------ */
/**
 * Starts to countdown.
 * @public
 * @emits Countdown#start
 */
const start = () => {
  if (counting.value) {
    return;
  }

  counting.value = true;

  if (!props.autoStart) {
    totalMilliseconds.value = time.value;
    endTime.value = props.now() + time.value;
  }

  if (props.emitEvents) {
    /**
     * Countdown start event.
     * @event Countdown#start
     */
    emit(EVENT_START);
  }

  if (document.visibilityState === "visible") {
    continue_();
  }
};

/**
 * Continues the countdown.
 * @private
 */
const continue_ = () => {
  if (!counting.value) {
    return;
  }

  const delay = Math.min(totalMilliseconds.value, props.interval);

  if (delay > 0) {
    let init: number;
    let prev: number;
    const step = (now: number) => {
      if (!init) {
        init = now;
      }

      if (!prev) {
        prev = now;
      }

      const range = now - init;

      if (
        range >= delay ||
        // Avoid losing time about one second per minute (now - prev ≈ 16ms) (#43)
        range + (now - prev) / 2 >= delay
      ) {
        progress();
      } else {
        requestId.value = requestAnimationFrame(step);
      }

      prev = now;
    };

    requestId.value = requestAnimationFrame(step);
  } else {
    end();
  }
};

/**
 * Pauses the countdown.
 * @private
 */

const pause = () => {
  cancelAnimationFrame(requestId.value);
};

/**
 * Progresses to countdown.
 * @private
 * @emits Countdown#progress
 */

const progress = () => {
  if (!counting.value) {
    return;
  }

  update();

  if (props.emitEvents && totalMilliseconds.value > 0) {
    /**
     * Countdown progress event.
     * @event Countdown#progress
     */
    emit(EVENT_PROGRESS, {
      years: years.value,
      days: days.value,
      hours: hours.value,
      minutes: minutes.value,
      seconds: seconds.value,
      milliseconds: milliseconds.value,
      totalDays: totalDays.value,
      totalHours: totalHours.value,
      totalMinutes: totalMinutes.value,
      totalSeconds: totalSeconds.value,
      totalMilliseconds: totalMilliseconds.value,
    });
  }

  continue_();
};

/**
 * Aborts the countdown.
 * @public
 * @emits Countdown#abort
 */
const abort = () => {
  if (!counting.value) {
    return;
  }

  pause();
  counting.value = false;

  if (props.emitEvents) {
    /**
     * Countdown abort event.
     * @event Countdown#abort
     */
    emit(EVENT_ABORT);
  }
};

/**
 * Ends the countdown.
 * @public
 * @emits Countdown#end
 */
const end = () => {
  if (!counting.value) {
    return;
  }

  pause();
  totalMilliseconds.value = 0;
  counting.value = false;

  if (props.emitEvents) {
    /**
     * Countdown end event.
     * @event Countdown#end
     */
    emit(EVENT_END);
  }
};

/**
 * Updates the count.
 * @private
 */
const update = () => {
  if (counting.value) {
    totalMilliseconds.value = Math.max(0, endTime.value - props.now());
  }
};

/**
 * Restarts the count.
 * @public
 */
const restart = () => {
  pause();
  totalMilliseconds.value = time.value;
  endTime.value = props.now() + time.value;
  counting.value = false;
  start();
};

/**
 * visibility change event handler.
 * @private
 */
const handleVisibilityChange = () => {
  switch (document.visibilityState) {
    case "visible":
      update();
      continue_();
      break;

    case "hidden":
      pause();
      break;

    default:
  }
};
/**------------------- */

/**Slot Props -------- */
const transformedProps = computed(() =>
  props.transform({
    years: years.value,
    days: days.value,
    hours: hours.value,
    minutes: minutes.value,
    seconds: seconds.value,
    milliseconds: milliseconds.value,
    totalDays: totalDays.value,
    totalHours: totalHours.value,
    totalMinutes: totalMinutes.value,
    totalSeconds: totalSeconds.value,
    totalMilliseconds: totalMilliseconds.value,
  })
);
/**------------------- */
</script>
