# Can You Hear Sound Through a Video?

**Course:** COSC 073 — Computational Photography  
**Term:** Spring 2026  
**Author:** Kasuti Makau

---

## The Idea

What if you could figure out what sound was playing in a room just by watching a video of it — without ever listening to the audio?

That's what this project does. Using nothing but the video frames from a camera, we recover the pitch of a tone playing in the room. No microphone. No audio track. Just pixels.

---

## How Cameras Make This Possible

Most modern cameras don't take a photo all at once. They read the image one row of pixels at a time, starting from the top and working down to the bottom. By the time the camera finishes reading the last row, a tiny sliver of time has passed since it read the first row.

That tiny time gap turns out to be useful. If a speaker is vibrating while the camera reads the frame, the speaker cone moves a little bit between each row. So the top rows see the speaker in one position, and the bottom rows see it in a slightly different position. That pattern of movement — encoded row by row — contains the vibration frequency.

Think of it like this: if you dragged a pen across a spinning record while someone played a note, the pen would draw a wave on the record. The wave's frequency matches the note. The camera does the same thing, but with light instead of ink.

---

## The Setup

To make the vibration visible, I put a strip of **black electrical tape** on the edge of a speaker cone. The tape creates a sharp black-and-white edge that the camera can track precisely.

**Equipment:**
- Speaker playing a pure tone from a tone generator app (a fixed, clean pitch)
- Cameras mounted on a tripod so only the speaker moves — camera shake adds noise
- Two cameras were used for two different approaches:

| | **Trial 1–4** | **Trial 5** |
|---|---|---|
| Camera | Canon Rebel T5i DSLR | iPhone 16 Pro Max |
| Mode | Regular video, 30 fps | Slow-motion, 240 fps |
| How the signal works | Within a single frame's row-by-row scan | Speaker movement between frames |
| Frequencies tested | 200, 440, 880, 1200 Hz | 50, 60, 70 Hz |

Both cameras use CMOS sensors, which means both have a rolling shutter — rows are read one at a time, not all at once. That's the property the whole method relies on.

A "silence" clip was also recorded for each setup with no tone playing. This is used as a baseline to confirm that any signal we find is actually from the sound and not from camera shake or other noise.

---

## How the Analysis Works

1. **Pull out the frames** — extract individual images from the video file.

2. **Find the tape edge** — for every row of pixels in a frame, find the exact column where the tape edge sits (where brightness changes most sharply). This gives a list of numbers: one edge position per row.

3. **Look for a pattern** — plot those edge positions from top to bottom. If a tone is playing, you should see the edge wiggle back and forth in a wave pattern. Silence should look flat.

4. **Find the frequency** — run a Fourier transform (FFT) on that wiggle. The FFT answers: "how fast is this wave oscillating?" That number is the frequency — the pitch of the sound.

5. **Calibrate** — the FFT output needs to be scaled to real Hz. We figure this out by filming a known frequency first (440 Hz for the Canon, 50 Hz for the iPhone) and using it as a reference to tune the scale.

6. **Stack multiple frames** — one frame gives a rough answer. Using 10–25 frames stacked together sharpens the result, letting us tell the difference between nearby frequencies like 50 Hz and 60 Hz.

---

## What We Found

### It works for the calibration frequencies

| Camera | Sound played | What we recovered | Error |
|---|---|---|---|
| iPhone 16 Pro Max (240 fps) | 50 Hz | **50 Hz** | **0%** |
| Canon Rebel T5i DSLR (30 fps) | 440 Hz | **440 Hz** | **0%** |

At these frequencies, the match is perfect. The signal was very clean — about **47 dB above the noise floor**, which is a strong, clear signal.

### It breaks for other frequencies — here's why

| Camera | Sound played | What we recovered | Why it's wrong |
|---|---|---|---|
| iPhone | 60 Hz | ~50 Hz | The speaker naturally vibrates strongest at its own resonant frequency (~50 Hz), regardless of what tone you feed it |
| iPhone | 70 Hz | ~50 Hz | Same reason |
| Canon | 200 Hz | 440 Hz | The speaker's resonance at ~440 Hz overpowers the actual tone |
| Canon | 880 Hz | 440 Hz | Same — the speaker always "wants" to ring at 440 Hz |
| Canon | 1200 Hz | 440 Hz | Same |

The real problem wasn't the cameras — it was the speaker. Every speaker has a natural resonant frequency, like how a wine glass rings at a specific note when you tap it. When you drive the speaker with any other frequency, it still vibrates hardest at its own resonant pitch. The camera faithfully detected that resonance — just not the tone we intended to measure.

Using a tweeter (a speaker designed for high frequencies, with a much higher resonant frequency) would likely fix this and allow the method to work across a wider range of tones.

### A surprise about the Canon's readout speed

The Canon Rebel T5i DSLR shoots at 30 fps with 1080 rows per frame. A rough guess would put the row readout speed at 30 × 1080 = 32,400 rows per second. But when we calibrated using the 440 Hz clip, the actual speed came out to about **95,040 rows per second** — roughly 3× faster than expected.

This is normal: cameras read rows much faster than the frame rate suggests because a lot of the process runs in parallel. The exact speed isn't written in any spec sheet, which is why calibrating on a known frequency is essential.

### How high could we theoretically go?

The maximum frequency the method can detect is half the row readout speed (this is called the Nyquist limit):

| Camera | Row readout speed | Max detectable frequency |
|---|---|---|
| iPhone 16 Pro Max (slow-mo) | 6,750 rows/sec | 3,375 Hz |
| Canon Rebel T5i DSLR | 95,040 rows/sec | 47,520 Hz |

In practice we recovered far less than this — the speaker's resonance was the bottleneck, not the cameras.

---

## Conclusion

Both cameras can, in principle, detect sound purely from video. The iPhone's 240 fps slow-motion mode and the Canon's 30 fps rolling shutter are different mechanisms but both worked for their calibration frequency with 0% error. The fundamental limitation was the speaker: its mechanical resonance drowned out all other frequencies. With a better speaker, the method should work across a much wider range of pitches.
