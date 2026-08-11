# Calibrating your own fan

Version 1.0.0

The percentages shipped in the package were measured on one fan in a 197 mm duct
with a specific run of ducting. Yours will differ. This is how to find your own
numbers in about twenty minutes.

## 1. Find the start and stop thresholds

Temporarily set `min_power: 0.0` on the LEDC output and flash.

Step the speed up from 0 in 1 % increments until the fan starts. Note the value.
Then step back down until it stops. Note that too — it will be lower. That gap is
real hysteresis in the motor controller, not a measurement error.

Set `min_power` to the *start* threshold, expressed as a fraction. If your fan
starts at 32 % duty, use `min_power: 0.32`. This maps a 1 % command onto the point
where the fan actually begins to move, so the low end of the slider does something.

## 2. Map the shelf

With `min_power` set, sweep from 1 % to 100 % and record airflow at each step. An
anemometer at the duct outlet is the honest measurement; a vane meter or even a
consistent tissue-deflection test will find the shape.

You are looking for where the flat region ends. Most of these fans have a plateau
at the bottom where the motor is at its floor and additional duty does nothing,
then a ramp. On the reference fan the shelf ran to about 25 % and the ramp took
over from there.

There is no fine control within the shelf. That is a limit of the fan, not
something a configuration change will recover.

## 3. Find the quiet knee

If noise matters more than airflow, sweep upward with a sound meter and find the
highest percentage before the first step increase in dB. On the reference fan that
was 31 %, with 32 % jumping about 3 dB. Set your quiet preset there rather than
guessing low — you are giving up airflow for nothing below the knee.

A phone sound meter app is adequate for finding the knee. You are looking for the
inflection, not an absolute SPL.

## 4. Set your presets

Edit the `preset_modes` list and the matching speeds in `on_preset_set`. Both need
to change — the list is the label, the automation is the value. Keep them in sync
or a preset will silently do nothing.

## 5. If you want to try the tach

The tach block in the package is commented out because it did not produce
trustworthy numbers here. If you want to attempt it:

1. Confirm the signal exists. With the fan at full, put a meter in frequency mode
   on the white wire. Expect something in the low hundreds of Hz. If there is nothing at
   the pin, stop — the problem is wiring, not configuration.
2. Add an external 10 k pull-up from the tach GPIO to 3.3 V. The ESP's internal
   pull-up is around 45 k, too weak for a harness-length run.
3. Route the tach wire physically away from the PWM wire, and twist it with a
   ground return if the run is long. Most of the trouble here is PWM coupling.
4. Uncomment the sensor block and flash.
5. Calibrate the multiplier. Run the fan at 100 %, read the raw pulse rate before
   any filter, and work out pulses per revolution against a known RPM — either the
   figure on the fan's spec sheet or an optical tachometer. Two pulses per
   revolution is the common case, which means `multiply: 0.5`.

Expect the reading to be worse at low speed than high. Low speed means a lower
duty cycle, which means more switching edges per unit time, which means more
coupling into the counter. At 100 % duty the output is nearly always on and the
signal is comparatively clean.

If the number matters to you, measure airflow directly instead. The tach is a
monitor at best.
