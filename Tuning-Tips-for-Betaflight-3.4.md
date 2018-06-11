## Notes about some new 3.4 features and defaults

### I just want to fly, not read all this stuff...

No problem, flash the code, and do a brief test fly with the stock defaults.  **Both 8k and 32k gyro modes should fly on defaults**, though 32k may require more filtering - always check motor temps after the first flight.

Note that 3.4 contains major changes to filtering and several new features that dramatically improve potential flight performance.  Do not expect it to fly the same as earlier versions!

If motor temps are OK, and handling generally alright, I'd recommend pasting into the CLI:

```
set iterm_relax = RPY
```

This will enable the term_relax code, which markedly reduces I bounce-back after flips or rolls, and allows higher levels of I than before.  Typically I can be increased by 50% or more to improve directional stability while flying in turbulent air or when approaching gates at high speed.

```
set rc_smoothing_type = FILTER
```

This activates **low-pass filter based RC input smoothing**, which removes spikes and sharp edges that otherwise arise during rapid stick inputs.  It should keep the motors sounding smooth and running cool during frequent rapid stick inputs, without delaying responsiveness as much as the older interpolation method (which is still available as an alternative).  

Note that with any RC smoothing, the normal spikes from D weight or throttle inputs will be trimmed smooth.  While this improves efficiency, motor temperature, and smoothness of motor control signals, losing those spikes reduces the effectiveness of D weight, P, and throttle, during very rapid stick inputs.  **After enabling RC smoothing, if you notice a small reduction in response to rapid stick inputs, consider increasing D weight and P by up to about 20%**. 

### The defaults are OK but I want to make some changes.  What do I need to know?

**The new default D weight value is 0.6**.  This is approximately equal to 0.8 in pre-3.4 versions.  More D weight means greater immediacy of stick responses, particularly to quick stick movements.  If the default of 0.6 doesn't feel responsive enough at the same rates as before, try a higher value.  1.0 is sufficient to overcome the normal damping behaviour that D itself would otherwise slow down responses to your stick inputs - the quad shifts from measurement to error mode of D calculation.  Values above 1.0 provide an additional 'feed forward' effect.  Too much D weight can feel excessively twitchy. 

**The new default D Setpoint Transition value is zero**.  If you previously flew with 1.0 or 0.5, to get a smooth centre feel, and it now feels too twitchy around centre sticks, try your old setting.  The default of 0 provides equal stick responsiveness regardless of stick position, and is recommended for racing.  Values under 0.1 are not recommended.

If your PID settings were higher than the current defaults, and the quad feels like it is a bit less responsive than before, try with values for P, D, and D weight more like what you had, and also try 20% above that. 

## Can I enable the new features on F3 boards?

A custom build with some features disabled can be made that will run on F3's with more than 128k of flash.  For instance I can make a build that will run 4k4k on a MOTOLAB F3 at 30% CPU with dual filters, throttle boost, iTerm_relax and filter based rc_smoothing.

### What about these new dual filters?

3.4 provides dual low-pass filter capability for both gyro and D filtering.   The default is to have all four low-pass filters active at the same time, the dynamic notch filter on, but no other notch filters active.  This provides less filtering delay, typically, than before, but with better filter performance.  

The two gyro filters clean up noise before the gyro signal enters the PID loop.  P, I and D are then derived from that filtered data.  The two D filters are applied only to the D signal.

To determine the relative contribution of P and D noise to overall motor noise, analyse a blackbox spectrum from PID_P and compare that to a spectrum from PID_D.  Typically there will be more D noise than P noise.  Hence we usually need filter D more heavily than P. 

The more filtering we apply, the less noise gets through to the motors, and this keeps them sounding smooth and running cool.  If we apply too much filtering, the PID calculations will be delayed, and flight performance will suffer.  Without enough filtering, the motors may run hot, especially if the props get bent or the motor bearings are worn, etc.

Dual PT1 filters allow fine tuning of the amount of filtering to match the noisiness of the quad.

Here are some examples of typical filtering setups, that can be pasted into the CLI:

A really good quad with a very solid frame, new motors, and only ever flown with clean well balanced light props:

```
set gyro_lowpass_type = PT1
set gyro_lowpass_hz = 150
set gyro_lowpass2_type = PT1
set gyro_lowpass2_hz = 400
set dterm_lowpass_hz = 100
set dterm_lowpass2_hz = 250
```

A typical 5" quad with motors/props that aren't new but are average, and bent props aren't a problem: 
(if MPU6000 gyro is used `gyro_lowpass2` can be set to 0 for best performance but logging/hot motors should be used to verify if this ok)

```
set gyro_lowpass_type = PT1
set gyro_lowpass_hz = 120
set gyro_lowpass2_type = PT1
set gyro_lowpass2_hz = 300
set dterm_lowpass_hz = 100
set dterm_lowpass2_hz = 250
```
A typical quad on a decent frame that needs to tolerate bent props:

```
set gyro_lowpass_type = PT1
set gyro_lowpass_hz = 120
set gyro_lowpass2_type = PT1
set gyro_lowpass2_hz = 300
set dterm_lowpass_hz = 80
set dterm_lowpass2_hz = 160
```

A beat up quad on a sloppy frame that often is flown with bent props:

```
set gyro_lowpass_type = PT1
set gyro_lowpass_hz = 90
set gyro_lowpass2_type = PT1
set gyro_lowpass2_hz = 180
set dterm_lowpass_hz = 70
set dterm_lowpass2_hz = 140
```

The higher the filter numbers - meaning, less filtering - and the better the state of the quad, the better it will fly.

## Should I keep the dynamic notch filter on always?

Probably.  It does add delay, but really helps if a prop gets bent.  For super clean setups where performance is everything, try with it off.  Make a very heavily D filtered profile to limp home if needed, otherwise motors can overheat a lot.

## Do I need the fixed notch filters anymore?

Short answer:  No.  They cause a lot of delay, and dual PT1 filters usually are enough.

Long answer: Fixed notch filters may be useful if a log spectrum shows a clear noise peak despite the dynamic notch.  Typically a problematic peak will appear at prop resonant frequency on flexible frames.  The only way to know for sure is to get a blackbox log and use PID-Analyzer or Blackbox Explorer to perform spectral analysis.  Prop resonant frequency can be determined using an audio spectrum analyser and 'plucking' the propeller, sometimes just setting a D notch at that frequency can be useful.  

## Filter based RC smoothing 

3.4 brings, thanks to eTracer, RC smoothing, with less delay and less jitter than interpolation methods.  Furthermore it is capable of adjusting the filter setpoint automatically to suite the RC command intervals.  

To activate the filter method, paste and save ```set rc_smoothing_type = FILTER``` in the CLI.

A biquad filter is then applied to the set incoming RC data, smoothing off the sharp square corners at the default frequency.  Sharp steps on throttle, P and D weight are then attenuated.  Because D setpoint weight adds even bigger spikes with each RC input, a second filter is applied specifically to D setpoint weight.

The end result is a smoother set of motor traces.  Each incoming RC step results in an immediate step up in PIDsum and the motors, but the step up is not a sharp spike any more, reducing motor heat accumulation, improving efficiency, and making them sound smoother during rapid stick movements.  Spikes also can trigger iterm_windup protection, this code avoids that problem.

When ```set rc_smoothing_input_hz = 0```, the code calculates the ideal low-pass value for RC command based on the initial RC interval.  The same applies for ```set rc_smoothing_derivative_hz = 0```, which sets the second low-pass value that gets applied to D weight automatically.  For 9ms Sbus, the automatic value is 50Hz. The actual values are written into the blackbox log header.

If you run very high levels of D weight, a somewhat lower derivative smoothing frequency may smooth out some of the big spikes that you might otherwise see.

Some people have voiced concern that low-pass filtering RC inputs will cause delay.  While that is true, without any filtering the motors trace gets filled with spikes that appear at each RC input step.  The current filter based smoothing code just takes the sharp point off these spikes, and doesn't delay the bulk of the step, or the majority of the D weight effect at all - the step up in the motors trace is still there at exactly the same time, just a bit smoother and rounder.

## iTerm Relax

The new iTerm_relax functionality by JoeLucid cuts, or reduces, I accumulation during fast stick inputs.  It can be enabled to work on pitch and roll alone, or pitch, roll and yaw.

The code has two operational modes, setpoint and gyro.  Gyro mode is more clinical and complete in suppressing iTerm, Setpoint mode is a bit smoother with slightly softer landings after flips.

Setpoint mode applies a high-pass filter to RC input, resulting in a value that gets higher whenever the sticks are moved quickly.  When the rate of change is zero (i.e., the sticks are not moving), iTerm accumulation is normal.  Accumulation is then attenuated linearly as the stick movement approaches a threshold.  Above threshold, no iTerm accumulation occurs at all.  

Gyro mode uses a high-pass filter based on rate of change of stick movement, and uses this to create a window either side of the gyro value inside which the quad should be tracking.  While inside the window, no iTerm accumulation occurs.  If the sticks are held still, the window compresses back to nothing, and iTerm accumulation becomes normal again.

Since Gyro bases the amount of iTerm suppression on how well the quad follows the expected path it can adjust the iTerm if that's required to implement the setpoint change demanded by the sticks. This can facilitate better tracking in unbalanced quads or if throttle changes are applied during the move.

In both case the filter time constant can be adjusted to fine-tune the response.  Typically no adjustment is required.  Sometimes after a fast multi-rotation flip, or other fast input, some I may accumulate and hang around too long once the stick movements stop, causing a late settling of turn rate.  Typically this will be less of a problem if the frequency is increased.

This feature is really useful and is capable, in a quad where P and D are well tuned, of totally eliminating bounce-back due to I.

## Throttle Boost

This feature, also by JoeLucid, improves responsiveness to fast throttle inputs.  The code uses high-pass filtering on the RC throttle signal to create additional 'kicks' in throttle.  It is basically a feed forward factor on throttle.  The faster the throttle stick is moved, the greater the boost effect.  The result is a quicker, more responsive feel to fast throttle inputs - both on increasing throttle, and cutting throttle.  It is particularly useful for quads that are weaker in terms of throttle responsiveness, or for making 3S feel like 4S, or 4S feel like 5S.

The height of each 'kick' is determined by the size of the RC step multiplied by the ```throttle_boost``` amount, and how long it lasts is set by the ```throttle_boost_cutoff``` value.  

The default throttle_boost_cutoff value is 15Hz, meaning a time constant of 10ms.  This works well for SBus 9ms radio setups, with or without rc smoothing on throttle.  It results in about 40% residual 'kick' carrying over into the next step, accumulating as the stick movement continues quickly, and dissipating otherwise.  The kick is rounded somewhat.  

If rc_smoothing is enabled on throttle, a higher cutoff frequency can reduce the residual for accumulation, allowing the rc smoothing handles the spikes that would otherwise appear.  For example, a value of 25 at 50Hz is about the same as 5 at 15Hz, but with less overshoot tendency.

If the throttle_boost value is 0, the effect is disabled.

With the default throttle_boost_cutoff of 15, throttle_boost = 5 is enough to overcome most to all RC smoothing delay on throttle (if indeed rc smoothing was applied to throttle).  Throttle should feel a bit more responsive - and sound smoother - than un-smoothed throttle input.  Increasing boost above 5 at 15Hz results in feed forward on throttle.  Values of 10 or higher can be used to overcome excessive motor delay when over-propping weaker motors.  Too much throttle boost may cause a kind of over-run and excessive sensitivity to big inputs.  This can be attenuated by choosing a higher throttle boost cutoff frequency, e.g. 50Hz, and a corresponding increase in amount, e.g. 25.  RC smoothing on throttle then becomes essential.  That's because higher cutoff frequency will cause sharper spikes of shorter duration, with less tendency to accumulate.  

Leaving the default value at 15 is probably a good idea. Better, if a sensitivity adjustment is needed, to change the throttle_boost value.

## iTerm Rotation

This is also by JoeLucid and is activated by default and much appreciated by LOS acro pilots, particularly when yawing continuously during rolls and when performing funnels and other tricks.  The code rotates the current iTerm vector properly as the quad rotates on other axes.  For FPV the effect is fairly subtle but can result in somewhat more predictable responses during abrupt stick inputs and while performing tricks.  There are no settings to adjust, just on or off.

## Smart Feed Forward

This experimental code, again by JoeLucid, modifies how D weight works.  It is not enabled by default  

Normally, D setpoint weight is a feed forward amount that increases with quicker RC stick movements. The greater the D weight, the greater the stick sensitivity to pitch and roll.

In the classical betaflight PID system, D setpoint weight assists P in initiating turns.  Typically D setpoint weight climbs quicker and relatively earlier than P.  Adding more D setpoint weight, and reducing P a little, can reduce PID overshoot without losing stick sensitivity.

Smart Feed forward changes the D setpoint weight behaviour, such that it replaces P entirely, but only when it is greater than P (and in the same direction).  For it to work, the D setpoint value must be set significantly higher than usual - up to 2.0 or more.  Then, a large part of the initial part of a turn is driven very hard by D weight, and P doesn't need to do so much.  In some settings, this can reduce overshoot a bit.   

## Absolute Control

This is experimental code by JoeLucid as well.  It's purpose is to address the same attitude issues as iterm_rotation but without some of its downsides.

Absolute Control continuously measures the error of the quads path over stick input, properly rotated into the quads coordinate system, and mixes a correction proportional to that error into the setpoint. It's as if you noticed every tiny attitude error the quad incurs and provided an instantaneous correction on your TX.

The result is significantly better tracking to sticks, particularly during rotations involving yaw and other difficult situations like throttle blips.

Absolute control will likely eventually replace iterm_rotation, but is not yet enabled by default. You should not enable abs_control and iterm_rotation at the same time.

To enable: ```set abs_control_gain = 10```.  Smaller quads are ok with 5.  

AbsoluteControl needs to be used with iTermRelax to avoid bounce-backs due to the latency between stick movement and quad response. iTermRelax will then suspend AbsoluteControl error accumulation as well during quick moves. Finally AbsoluteControl only kicks in once the throttle minimum for airmode activation is exceeded to avoid undue corrections on the ground. 

## My quad doesn't feel as responsive as it used to!

If you've enabled rc_smoothing and it is set to interpolation, try ```set rc_smoothing_type = FILTER```.  

Consider increasing P, I and D by around 20% over previous values.

If the quad didn't need much filtering, try the 'clean quad' filter settings, above.

If all else fails, revert to your old code - you may find, by now, that it doesn't feel quite as good as you remember it!