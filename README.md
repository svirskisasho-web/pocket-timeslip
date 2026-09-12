# Pocket Timeslip

A GPS + accelerometer drag timer that runs in a phone browser. Measures 0–100 km/h,
400 m, the quarter mile and the usual strip splits, with trap speeds.

**Live:** https://svirskisasho-web.github.io/pocket-timeslip/

## How it times

The browser only gives GPS about **once per second**, which is far too coarse on its
own. So the clock runs on the phone's accelerometer at 60–100 Hz, and every GPS fix
corrects that estimate's speed and accumulated drift:

- **Launch detection** comes from the accelerometer, with a backtrack through a ring
  buffer to the instant the car actually started moving — the start line is sharp to
  roughly ±30 ms.
- **The forward axis is learned at launch**, so the phone can be mounted in any
  orientation as long as it is rigid relative to the car.
- **Gravity is re-learned continuously while stationary**, and its component is
  stripped out of each reading, so bumps and body pitch matter less.
- **Speed is anchored by GPS Doppler** (`coords.speed`), which is the genuinely
  accurate part of a GPS fix, with a complementary filter estimating accelerometer
  bias. Fix timestamps are mapped into the `performance.now()` domain so the
  correction is applied against the fused speed at the moment the fix was taken,
  not the moment it was delivered.

Expect within **±0.1–0.3 s** of a 10 Hz Dragy box on 0–100, and better than that
run-to-run. Good for A/B testing a map, fuel or tyres. Not a timing certificate.

## Notes

- Needs to be served over HTTPS (or opened as a local `file://`) as a **top-level
  page**. Sensors are blocked inside cross-origin iframes.
- Runs are stored in `localStorage` on the phone — nothing leaves the device, and it
  works with no signal once loaded. Export is via clipboard, not download.
- Rollout (1 ft) is a toggle in Setup, off by default.
- Hold the phone rigidly. Loose on the seat and the fusion is worthless.

## Using it

Open the page, tap **Start GPS**, grant location. Wait for the accuracy chip to go
green (under 5 m — give it 2–3 minutes outdoors on first use). Come to a complete
stop, tap **Arm**, hold still for a second while it calibrates, then launch. It stops
itself past the last distance marker or when you roll to a halt.
