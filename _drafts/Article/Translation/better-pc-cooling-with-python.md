---
title: Better PC cooling with Python and Grafana
authorURL: ""
originalURL: https://calbryant.uk/blog/better-pc-cooling-with-python
translator: ""
reviewer: ""
---

[Cal Bryant][1]

<!-- more -->

-   [Home][2]
-   [About][3]
-   [etc][4]

Table of Contents

-   [The idea][5]
-   [Goals][6]
-   [Research][7]
    -   [Liquidctl][8]
    -   [lm-sensors][9]
    -   [sysfs/hwmon][10]
-   [The solution][11]
    -   [Calibration][12]
    -   [The control loop][13]
    -   [Installation][14]
-   [Measuring performance with Grafana][15]
    -   [Installation & setup][16]
    -   [Terminology][17]
    -   [Recording][18]
-   [Results][19]
-   [Conclusion][20]
-   [Future improvements][21]
    -   [Hybrid mode][22]
    -   [Abstraction][23]
    -   [Integrated monitoring][24]
    -   [Stall speed detection][25]
    -   [Beat frequencies][26]
    -   [General undervolting][27]
    -   [Better fans and thermal interface compound][28]

# Better PC cooling with Python and Grafana

Mar 2024 - 16 min read

I recently upgraded from a [Ryzen 3700X to a 5950X.][29] Double the cores, and nearly double the potential heat output. I didn’t upgrade my cooling solution, [a 240mm Kraken X53][30] AIO liquid cooler.

Doing any real work with the 5950X made my PC significantly louder, and worse yet the fans were now spinning up and down suddenly and erratically.[1][31]

[![Inside -- the Kraken X53 and rear case fan, 92mm](/blog/better-pc-cooling-with-python/IMG_8253_hue35ee26d784f96ecdda6bf220ed6ef9a_855530_240x240_fit_q90_lanczos.JPEG)][32]

Inside -- the Kraken X53 and rear case fan, 92mm

[![The 2 Noctua 120mm fans on the radiator](/blog/better-pc-cooling-with-python/IMG_8255_huf307f0286bd92bd70992b423eb0d0be6_862441_240x240_fit_q90_lanczos.JPEG)][33]

The 2 Noctua 120mm fans on the radiator

[![Assembled, blasted with my compressor.](/blog/better-pc-cooling-with-python/IMG_8256_huee3d40fa01e559d832322055019e58b4_1076509_240x240_fit_q90_lanczos.JPEG)][34]

Assembled, blasted with my compressor.

The reason for this is the radiator fans are controlled based on the CPU temperature, which quickly ramps up and down itself. This is the only option using the motherboard based fan control configurable in the UEFI for me – the X53 cannot control fans by itself.

I presume the quick temperature rises are specific to modern Ryzen CPUs, perhaps others too. Maybe this is due to more accurate sensors, or even a less-than-ideal thermal interface. Right now, I’m uncertain it’s not my thermal compound even.

I know modern CPUs – particularly Ryzen 5000/7000 or intel 13th/14th gen – are designed to boost as much as possible with tight margins around temperature and power limits.[2][35]

The kraken cooler is by default designed to vary the pump speed based on liquid temperature. I think this is not optimal for cooling – it does reduce the slight whine of the pump however.

## The idea

As I use liquid cooling, there’s significant thermal mass available which really should mean the sudden ramping behaviour of the fans is not required.

If I could instead control the pump speed based on CPU temperature and the fan speed based on liquid temperature, I could take advantage of the thermal mass of the liquid to avoid ramping up the fans unnecessarily as the liquid takes some time to heat.

The CPU would also be cooled more effectively, and the rate of heat transfer to the liquid would peak with the CPU demand, instead of being tied to liquid temperature.

## Goals

-   Reduce irritating erratic fan speeds
-   Reduce noise
-   Reduce dust
-   Eliminate throttling if any
-   Work on NixOS (my main OS)

[![A bit of dust. Let's limit that build up. Eww.](/blog/better-pc-cooling-with-python/dust_hu65376ab7cfca78a136649b0ef87e58a5_1159388_240x240_fit_q90_lanczos.jpeg)][36]

A bit of dust. Let's limit that build up. Eww.

Whilst I’m at it I may as well attempt a negative [PBO2 offset][37] to reduce the heat output of the CPU, and apply better thermal interface material in the hope to make cooling more effective. I could also try a traditional underclock/undervolt as described [here][38].

## Research

I decided to write a python script, installed as a [systemd service][39] to implement the idea. I’d need to read CPU temperature + liquid temperature, and control fan + pump speed.

### Liquidctl

[Liquidctl][40] is an excellent project that allows programmatic control of the X53 among others. It even has python bindings! Writing the control loop in python therefore seemed like a good choice.

Liquidctl with the X53 allows reading & controlling pump speed as well as liquid temperature; unfortunately the X range of Krakens does not allow radiator fan speed control unlike the Z series. I had to find a way of controlling the radiator fans and also reading the CPU temperature.

For controlling the fans I considered making my own fan controller PCB, or using a [Corsair Commander][41] which I know can be interfaced under linux, also with liquidctl.

### lm-sensors

In the mean time, I looked at [lm-sensors][42] has been around since the dawn of time. It is able to interrogate a plethora of hardware sensors by scanning various busses and integrating with many kernel modules. There are python bindings too.

[![Truncated lm-sensors output](/blog/better-pc-cooling-with-python/sensors_hu2b54c135aacf3b9e9633566644fbce07_32757_1280x720_fit_q90_bgffffff_lanczos_3.jpg)][43]

Truncated lm-sensors output

I experimented with parsing the output, and using the module. This worked fine – it was a little awkward due to the nested tree structure from lm-sensors – but nothing a flattening couldn’t fix.[3][44] I didn’t like the extra complexity of the necessary `sensors-detect` scanning, nor the fact that I ended up calling the `lm-sensors` executable multiple times a second.

In the end I found a way of reading the temperature and controlling the fans connected to the motherboard using python after a friend suggested the possibility. This was thanks to the lm-sensors source code and [script][45]s – I was able to find a fan control bash script that appeared to be interfacing with [sysfs][46] directly.

### sysfs/hwmon

I figured I could do the same thing, and possibly read temperatures too! As it turns out, since `lm-sensors` `3.0.0` (2007) the kernel module drivers all implement the same `sysfs` interface and `libsensors` is an abstraction atop this to normalise manufacturer specific values.

[![hwmon tree](/blog/better-pc-cooling-with-python/hwmon_hu74cfcd43c8b53be4a50b555c9ba57ba6_32268_1280x720_fit_q90_bgffffff_lanczos_3.jpg)][47]

hwmon tree

This `sysfs` interface [is documented here][48]. It’s straightforward! Just writing and reading values from specific files.

The kernel module I had to load after running `sensors-detect` was `nct6775`. This is for a family of system management chips, one of which exists on my motherboard. After loading this module, I could interface via `sysfs` – without `libsensors` or `lm-sensors`; this is great news as it means my script can be much simpler with one less dependency. nct6775-specific settings are [documented here][49].[4][50].

I’m also going to use `k10temp` to get the best reading of temperature from the CPU directly.

Here’s a quick summary on the files used to interface with fans and sensors with sysfs. Substitute `hwmon5`, `pwm2` & `temp1` (etc) for your own controller and channels.

-   `/sys/class/hwmon/hwmon5/pwm2_enable` – manual: `1`, auto: `5`
-   `/sys/class/hwmon/hwmon5/pwm2` – value of PWM duty, `0` - `255`
-   `/sys/class/hwmon/hwmon1/temp1_input` – temp in °C
-   `/sys/class/hwmon/hwmon1/temp1_name` – name of temperature sensor
-   `/sys/class/hwmon/hwmon5/fan2_input` – measured fan speed in RPM
-   `/sys/class/hwmon/hwmon5/name` – name of controller

To find the right path for a given fan, you can look for clues via the given names and also figure out the mapping by writing values and watching the fans/sensors/temperatures change. Make sure you restore automatic mode after! Note that simply switching from automatic to manual is usually enough, as it will cause 100% duty and make it obvious what fan is connected.

## The solution

You can download the [complete script here][51]. I’ll explain the meat of how it works in this section. Be warned – the script is specific to my system, but could be adapted.

Note having the control loop running on my OS could be risky – if the control loop crashes, the CPU could overheat or damage could otherwise be caused. Bear this in mind. That said, the CPU is designed to throttle so in practice it would be difficult to cause any damage.

I also rely on systemd to restart the python application if it crashes. Crashing is detectable by checking `systemd` and observing the calibration cycle – the fans ramp up.

### Calibration

I use Noctua fans. According to the Noctua datasheets, Noctua guarantee the fans can run above a 20% duty cycle; below that is undefined. Usually, fans will run quite a bit below this before stopping – we should figure out what the minimum value is empirically on startup for the quietest experience possible.

<table class="w7"><tbody><tr><td class="sk"><pre class="chroma" tabindex="0"><code><span class="j4"> 1
</span><span class="j4"> 2
</span><span class="j4"> 3
</span><span class="j4"> 4
</span><span class="j4"> 5
</span><span class="j4"> 6
</span><span class="j4"> 7
</span><span class="j4"> 8
</span><span class="j4"> 9
</span><span class="j4">10
</span><span class="j4">11
</span><span class="j4">12
</span><span class="j4">13
</span><span class="j4">14
</span><span class="j4">15
</span><span class="j4">16
</span><span class="j4">17
</span><span class="j4">18
</span><span class="j4">19
</span><span class="j4">20
</span><span class="j4">21
</span><span class="j4">22
</span></code></pre></td><td class="sk"><pre class="chroma" tabindex="0"><code class="language-python" data-lang="python"><span class="line"><span class="f"><span class="q9">def</span> <span class="px">min_speed</span><span class="a">(</span><span class="n">pwm_file</span><span class="a">:</span> <span class="ao">str</span><span class="a">,</span> <span class="n">fan_input_file</span><span class="a">)</span> <span class="hb">-&gt;</span> <span class="ao">tuple</span><span class="a">[</span><span class="ao">int</span><span class="a">,</span> <span class="ao">int</span><span class="a">]:</span>
</span></span><span class="line"><span class="f">    <span class="uf"># Turn off fans, wait to stop</span>
</span></span><span class="line"><span class="f">    <span class="n">set_file_int</span><span class="a">(</span><span class="n">pwm_file</span><span class="a">,</span> <span class="r0">0</span><span class="a">)</span>
</span></span><span class="line"><span class="f">
</span></span><span class="line"><span class="f">    <span class="q9">for</span> <span class="n">x</span> <span class="d7">in</span> <span class="ao">range</span><span class="a">(</span><span class="r0">10</span><span class="a">):</span>
</span></span><span class="line"><span class="f">        <span class="n">sleep</span><span class="a">(</span><span class="dh">0.5</span><span class="a">)</span>
</span></span><span class="line"><span class="f">        <span class="q9">if</span> <span class="n">get_file_int</span><span class="a">(</span><span class="n">fan_input_file</span><span class="a">)</span> <span class="hb">==</span> <span class="r0">0</span><span class="a">:</span>
</span></span><span class="line"><span class="f">            <span class="q9">break</span>
</span></span><span class="line"><span class="f">    <span class="q9">else</span><span class="a">:</span>
</span></span><span class="line"><span class="f">        <span class="q9">raise</span> <span class="o5">RuntimeError</span><span class="a">(</span><span class="kg">"Could not stop fan for calibration"</span><span class="a">)</span>
</span></span><span class="line"><span class="f">
</span></span><span class="line"><span class="f">    <span class="uf"># ramp up fan, until it starts moving</span>
</span></span><span class="line"><span class="f">    <span class="q9">for</span> <span class="n">duty</span> <span class="d7">in</span> <span class="ao">range</span><span class="a">(</span><span class="r0">100</span><span class="a">):</span>
</span></span><span class="line"><span class="f">        <span class="n">set_file_int</span><span class="a">(</span><span class="n">pwm_file</span><span class="a">,</span> <span class="n">duty</span><span class="a">)</span>
</span></span><span class="line"><span class="f">        <span class="n">sleep</span><span class="a">(</span><span class="dh">0.5</span><span class="a">)</span>
</span></span><span class="line"><span class="f">
</span></span><span class="line"><span class="f">        <span class="q9">if</span> <span class="n">get_file_int</span><span class="a">(</span><span class="n">fan_input_file</span><span class="a">)</span> <span class="hb">&gt;</span> <span class="r0">0</span><span class="a">:</span>
</span></span><span class="line"><span class="f">            <span class="q9">break</span>
</span></span><span class="line"><span class="f">    <span class="q9">else</span><span class="a">:</span>
</span></span><span class="line"><span class="f">        <span class="q9">raise</span> <span class="o5">RuntimeError</span><span class="a">(</span><span class="kg">"Could not start fan for calibration"</span><span class="a">)</span>
</span></span><span class="line"><span class="f">
</span></span><span class="line"><span class="f">    <span class="q9">return</span> <span class="n">duty</span><span class="a">,</span> <span class="n">get_file_int</span><span class="a">(</span><span class="n">fan_input_file</span><span class="a">)</span></span></span></code></pre></td></tr></tbody></table>

[Copy][52]

The minimum duty cycle is actually hysteretical – the fan will run at a lower speed than the start speed if already running due to momentum. To be safe, I prefer no start from 0% duty and increment slowly until the fan starts so it will always recover from a stall – as above.

I discovered the fans can start at around 11% duty, 200RPM – almost half the guaranteed duty and less than half the min speed – great! This means less noise and dust. This calibration is performed automatically at start.

I also measure the maximum RPM for curiosity on startup – by setting the duty to 100% and waiting.

The CPU temperature range is based on the maximum temperature defined by AMD, and a measured idle temperature at max cooling; spoiler: this worked fine, without any adjustment.

The liquid temperature was chosen ranging from idle temperature to max temperature, both on full cooling. This seemed to work well, too. For both, my room temperature was around 20c.

As for the case temperature, I took some values I considered reasonable.

### The control loop

Most PC fan control software maps a temperature to a fan speed, using some kind of curve. For instance, 40-70c may correspond to 600-1500RPM. The hope is that, for a given heat output, the fan speed will settle at an equilibrium. This is done using the concept of simple [negative feedback][53].

[![Example fan curve from a friend's liquid cooler](/blog/better-pc-cooling-with-python/curve_hua1fe8a2f830643cc80f5ccb2836d6b77_49266_854x480_fit_q90_bgffffff_lanczos_3.jpg)][54]

Example fan curve from a friend's liquid cooler

Some curves may rise quickly – presumably to anticipate a load, or gradually, to slow down the initial response of the system to perhaps ride out small peaks in demand. The peaks could cause annoying fluctuations in speed, otherwise.[1][55]

[![The debug output of the first working version](/blog/better-pc-cooling-with-python/running_hu04a36e1436e3a1853e869bb65d6872a2_35807_854x480_fit_q90_bgffffff_lanczos_3.jpg)][56]

The debug output of the first working version

I know some BIOSes also allow a delay time constant to further smooth the response; basically a low pass filter simulating thermal mass!

**I think the best solution is to have actual thermal mass** – the liquid. This allows a smoother response without sacrificing cooling performance when it’s needed most. Especially important given how aggressively modern CPUs boost.

Anyway, the control loop reads 3 temperatures (liquid, case and CPU) then scales them linearly to 3 PWM duties – the pump, case fan and radiator fan. The PWM values are capped between the minimum PWM (detailed above) and 100%.

<table class="w7"><tbody><tr><td class="sk"><pre class="chroma" tabindex="0"><code><span class="j4"> 1
</span><span class="j4"> 2
</span><span class="j4"> 3
</span><span class="j4"> 4
</span><span class="j4"> 5
</span><span class="j4"> 6
</span><span class="j4"> 7
</span><span class="j4"> 8
</span><span class="j4"> 9
</span><span class="j4">10
</span><span class="j4">11
</span><span class="j4">12
</span><span class="j4">13
</span><span class="j4">14
</span><span class="j4">15
</span><span class="j4">16
</span><span class="j4">17
</span></code></pre></td><td class="sk"><pre class="chroma" tabindex="0"><code class="language-python" data-lang="python"><span class="line"><span class="f"><span class="q9">def</span> <span class="px">loop</span><span class="a">(</span><span class="nm">self</span><span class="a">):</span>
</span></span><span class="line"><span class="f">    <span class="ao">print</span><span class="a">(</span><span class="kg">"Entering main loop..."</span><span class="a">)</span>
</span></span><span class="line"><span class="f">    <span class="q9">for</span> <span class="n">status</span> <span class="d7">in</span> <span class="n">controller</span><span class="hb">.</span><span class="n">watch</span><span class="a">():</span>
</span></span><span class="line"><span class="f">        <span class="n">liquid_factor</span> <span class="hb">=</span> <span class="a">(</span><span class="n">status</span><span class="a">[</span><span class="kg">"liquid_temp"</span><span class="a">]</span> <span class="hb">-</span> <span class="n">LIQUID_BASE_TEMP</span><span class="a">)</span> <span class="hb">/</span> <span class="a">(</span>
</span></span><span class="line"><span class="f">            <span class="n">LIQUID_MAX_TEMP</span> <span class="hb">-</span> <span class="n">LIQUID_BASE_TEMP</span>
</span></span><span class="line"><span class="f">        <span class="a">)</span>
</span></span><span class="line"><span class="f">        <span class="n">cpu_factor</span> <span class="hb">=</span> <span class="a">(</span><span class="n">status</span><span class="a">[</span><span class="kg">"cpu_temp"</span><span class="a">]</span> <span class="hb">-</span> <span class="n">CPU_BASE_TEMP</span><span class="a">)</span> <span class="hb">/</span> <span class="a">(</span>
</span></span><span class="line"><span class="f">            <span class="n">CPU_MAX_TEMP</span> <span class="hb">-</span> <span class="n">CPU_BASE_TEMP</span>
</span></span><span class="line"><span class="f">        <span class="a">)</span>
</span></span><span class="line"><span class="f">
</span></span><span class="line"><span class="f">        <span class="n">case_factor</span> <span class="hb">=</span> <span class="a">(</span><span class="n">status</span><span class="a">[</span><span class="kg">"case_temp"</span><span class="a">]</span> <span class="hb">-</span> <span class="n">CASE_BASE_TEMP</span><span class="a">)</span> <span class="hb">/</span> <span class="a">(</span>
</span></span><span class="line"><span class="f">            <span class="n">CASE_MAX_TEMP</span> <span class="hb">-</span> <span class="n">CASE_BASE_TEMP</span>
</span></span><span class="line"><span class="f">        <span class="a">)</span>
</span></span><span class="line"><span class="f">
</span></span><span class="line"><span class="f">        <span class="n">controller</span><span class="hb">.</span><span class="n">set_pump_speed</span><span class="a">(</span><span class="n">cpu_factor</span><span class="a">)</span>
</span></span><span class="line"><span class="f">        <span class="n">set_fan_speed</span><span class="a">(</span><span class="n">RAD_PWM_FILE</span><span class="a">,</span> <span class="n">controller</span><span class="hb">.</span><span class="n">rad_fan_start_duty</span><span class="a">,</span> <span class="n">liquid_factor</span><span class="a">)</span>
</span></span><span class="line"><span class="f">        <span class="n">set_fan_speed</span><span class="a">(</span><span class="n">CASE_PWM_FILE</span><span class="a">,</span> <span class="n">controller</span><span class="hb">.</span><span class="n">case_fan_start_duty</span><span class="a">,</span> <span class="n">case_factor</span><span class="a">)</span></span></span></code></pre></td></tr></tbody></table>

[Copy][57]

This is done within a context manager to ensure we close the `liquidctl` device.

### Installation

As I mentioned I’d be running the fan control software as a `systemd` service, I figured it was worth detailing how – on NixOS – here. All that’s required is to add this snippet to `/etc/nixos/configuration.nix`. Super convenient!

<table class="w7"><tbody><tr><td class="sk"><pre class="chroma" tabindex="0"><code><span class="j4"> 1
</span><span class="j4"> 2
</span><span class="j4"> 3
</span><span class="j4"> 4
</span><span class="j4"> 5
</span><span class="j4"> 6
</span><span class="j4"> 7
</span><span class="j4"> 8
</span><span class="j4"> 9
</span><span class="j4">10
</span></code></pre></td><td class="sk"><pre class="chroma" tabindex="0"><code class="language-nix" data-lang="nix"><span class="line"><span class="f"><span class="n">systemd</span><span class="hb">.</span><span class="n">services</span><span class="hb">.</span><span class="n">fangoblin3</span> <span class="b3">=</span> <span class="a">{</span>
</span></span><span class="line"><span class="f"><span class="n">path</span> <span class="hb">=</span> <span class="a">[</span>
</span></span><span class="line"><span class="f">  <span class="a">(</span><span class="n">pkgs</span><span class="hb">.</span><span class="n">python3</span><span class="hb">.</span><span class="n">withPackages</span> <span class="a">(</span><span class="n">ps</span><span class="a">:</span> <span class="q9">with</span> <span class="n">ps</span><span class="a">;</span> <span class="a">[</span> <span class="n">liquidctl</span> <span class="a">]))</span>
</span></span><span class="line"><span class="f"><span class="a">];</span>
</span></span><span class="line"><span class="f"><span class="n">script</span> <span class="hb">=</span> <span class="kg">"exec </span><span class="rk">${</span><span class="e0">./fangoblin3</span><span class="rk">}</span><span class="ch">\n</span><span class="kg">"</span><span class="a">;</span>
</span></span><span class="line"><span class="f"><span class="n">wantedBy</span> <span class="hb">=</span> <span class="a">[</span> <span class="kg">"multi-user.target"</span> <span class="a">];</span>
</span></span><span class="line"><span class="f"><span class="n">serviceConfig</span> <span class="hb">=</span> <span class="a">{</span>
</span></span><span class="line"><span class="f">  <span class="n">Restart</span> <span class="hb">=</span> <span class="kg">"on-failure"</span><span class="a">;</span>
</span></span><span class="line"><span class="f">  <span class="n">RestartSec</span> <span class="hb">=</span> <span class="r0">10</span><span class="a">;</span>
</span></span><span class="line"><span class="f"><span class="a">};</span></span></span></code></pre></td></tr></tbody></table>

[Copy][58]

I hope you like the name of the script.

## Measuring performance with Grafana

### Installation & setup

I could have dumped the values via CSV and plotted graphs in a spreadsheet. However for multiple readings this can become tedious. I gave [Grafana][59], a monitoring solution combined with [influxDB][60] – a timeseries database. This is a common pairing.

I found 2 things non-intuitive when setting up this stack:

1.  Connecting the services together – terminology mismatch
2.  Influx-specific terminology around the data model
3.  Unhelpful error messages

…so I will cover setting the stack up and help make sense of it, as I presume someone else out there has faced similar difficulties. The “add data source” workflow and UI in Grafana looks polished, but in practice it does seem like a hack connecting the services together.

I used docker-compose to start influx and Grafana. Helpfully, you can initialise the database and set initial secrets as environment variables:

<table class="w7"><tbody><tr><td class="sk"><pre class="chroma" tabindex="0"><code><span class="j4"> 1
</span><span class="j4"> 2
</span><span class="j4"> 3
</span><span class="j4"> 4
</span><span class="j4"> 5
</span><span class="j4"> 6
</span><span class="j4"> 7
</span><span class="j4"> 8
</span><span class="j4"> 9
</span><span class="j4">10
</span><span class="j4">11
</span><span class="j4">12
</span><span class="j4">13
</span><span class="j4">14
</span><span class="j4">15
</span><span class="j4">16
</span><span class="j4">17
</span><span class="j4">18
</span><span class="j4">19
</span><span class="j4">20
</span><span class="j4">21
</span><span class="j4">22
</span><span class="j4">23
</span><span class="j4">24
</span><span class="j4">25
</span><span class="j4">26
</span><span class="j4">27
</span><span class="j4">28
</span><span class="j4">29
</span><span class="j4">30
</span><span class="j4">31
</span><span class="j4">32
</span><span class="j4">33
</span><span class="j4">34
</span><span class="j4">35
</span><span class="j4">36
</span><span class="j4">37
</span></code></pre></td><td class="sk"><pre class="chroma" tabindex="0"><code class="language-.yml" data-lang=".yml"><span class="line"><span class="f"><span class="rq">version</span><span class="a">:</span><span class="su"> </span><span class="d6">'2'</span><span class="su">
</span></span></span><span class="line"><span class="f"><span class="su"></span><span class="rq">services</span><span class="a">:</span><span class="su">
</span></span></span><span class="line"><span class="f"><span class="su">  </span><span class="rq">influxdb</span><span class="a">:</span><span class="su">
</span></span></span><span class="line"><span class="f"><span class="su">    </span><span class="rq">image</span><span class="a">:</span><span class="su"> </span><span class="d2">influxdb:latest</span><span class="su">
</span></span></span><span class="line"><span class="f"><span class="su">    </span><span class="rq">ports</span><span class="a">:</span><span class="su">
</span></span></span><span class="line"><span class="f"><span class="su">      </span>- <span class="d6">'8086:8086'</span><span class="su">
</span></span></span><span class="line"><span class="f"><span class="su">    </span><span class="rq">volumes</span><span class="a">:</span><span class="su">
</span></span></span><span class="line"><span class="f"><span class="su">      </span>- <span class="d2">influxdb-storage:/var/lib/influxdb2</span><span class="su">
</span></span></span><span class="line"><span class="f"><span class="su">    </span><span class="rq">environment</span><span class="a">:</span><span class="su">
</span></span></span><span class="line"><span class="f"><span class="su">      </span>- <span class="d2">DOCKER_INFLUXDB_INIT_MODE=setup</span><span class="su">
</span></span></span><span class="line"><span class="f"><span class="su">      </span>- <span class="d2">DOCKER_INFLUXDB_INIT_USERNAME=admin</span><span class="su">
</span></span></span><span class="line"><span class="f"><span class="su">      </span>- <span class="d2">DOCKER_INFLUXDB_INIT_PASSWORD=badpassword</span><span class="su">
</span></span></span><span class="line"><span class="f"><span class="su">      </span>- <span class="d2">DOCKER_INFLUXDB_INIT_ORG=default</span><span class="su">
</span></span></span><span class="line"><span class="f"><span class="su">      </span>- <span class="d2">DOCKER_INFLUXDB_INIT_BUCKET=default</span><span class="su">
</span></span></span><span class="line"><span class="f"><span class="su">      </span>- <span class="d2">DOCKER_INFLUXDB_INIT_RETENTION=4w</span><span class="su">
</span></span></span><span class="line"><span class="f"><span class="su">      </span>- <span class="d2">DOCKER_INFLUXDB_INIT_ADMIN_TOKEN=a0a77e5d-983d-4992-bcad-4a5280a9eca6</span><span class="su">
</span></span></span><span class="line"><span class="f"><span class="su">  </span><span class="rq">grafana</span><span class="a">:</span><span class="su">
</span></span></span><span class="line"><span class="f"><span class="su">    </span><span class="rq">image</span><span class="a">:</span><span class="su"> </span><span class="d2">grafana/grafana-enterprise</span><span class="su">
</span></span></span><span class="line"><span class="f"><span class="su">    </span><span class="rq">container_name</span><span class="a">:</span><span class="su"> </span><span class="d2">grafana</span><span class="su">
</span></span></span><span class="line"><span class="f"><span class="su">    </span><span class="rq">restart</span><span class="a">:</span><span class="su"> </span><span class="d2">unless-stopped</span><span class="su">
</span></span></span><span class="line"><span class="f"><span class="su">    </span><span class="rq">environment</span><span class="a">:</span><span class="su">
</span></span></span><span class="line"><span class="f"><span class="su">      </span><span class="rq">GF_RENDERING_SERVER_URL</span><span class="a">:</span><span class="su"> </span><span class="d2">http://renderer:8081/render</span><span class="su">
</span></span></span><span class="line"><span class="f"><span class="su">      </span><span class="rq">GF_RENDERING_CALLBACK_URL</span><span class="a">:</span><span class="su"> </span><span class="d2">http://grafana:3000/</span><span class="su">
</span></span></span><span class="line"><span class="f"><span class="su">      </span><span class="rq">GF_LOG_FILTERS</span><span class="a">:</span><span class="su"> </span><span class="d2">rendering:debug</span><span class="su">
</span></span></span><span class="line"><span class="f"><span class="su">    </span><span class="rq">ports</span><span class="a">:</span><span class="su">
</span></span></span><span class="line"><span class="f"><span class="su">      </span>- <span class="d6">'3000:3000'</span><span class="su">
</span></span></span><span class="line"><span class="f"><span class="su">    </span><span class="rq">volumes</span><span class="a">:</span><span class="su">
</span></span></span><span class="line"><span class="f"><span class="su">      </span>- <span class="d2">grafana-storage:/var/lib/grafana</span><span class="su">
</span></span></span><span class="line"><span class="f"><span class="su">
</span></span></span><span class="line"><span class="f"><span class="su">  </span><span class="rq">renderer</span><span class="a">:</span><span class="su">
</span></span></span><span class="line"><span class="f"><span class="su">    </span><span class="rq">image</span><span class="a">:</span><span class="su"> </span><span class="d2">grafana/grafana-image-renderer:latest</span><span class="su">
</span></span></span><span class="line"><span class="f"><span class="su">    </span><span class="rq">ports</span><span class="a">:</span><span class="su">
</span></span></span><span class="line"><span class="f"><span class="su">      </span>- <span class="sd">8081</span><span class="su">
</span></span></span><span class="line"><span class="f"><span class="su">
</span></span></span><span class="line"><span class="f"><span class="su"></span><span class="rq">volumes</span><span class="a">:</span><span class="su">
</span></span></span><span class="line"><span class="f"><span class="su">  </span><span class="rq">influxdb-storage</span><span class="a">:</span><span class="su">
</span></span></span><span class="line"><span class="f"><span class="su">  </span><span class="rq">grafana-storage</span><span class="a">:</span><span class="su">
</span></span></span></code></pre></td></tr></tbody></table>

docker-compose.yml [Download][61][Copy][62]

After a `docker compose up -d`, you can log in to the Grafana instance at `http://localhost:3000/` using `admin`/`admin`. After that you need to connect the Influx DB – do to `Home > Connections > Data sources` and click on InfluxDB after searching.[5][63]

[![Unhelpful error message](/blog/better-pc-cooling-with-python/influx1_hu9f183b9c68b128b845bbbd9f1b4f681e_161714_240x240_fit_q90_bgffffff_lanczos_3.jpg)][64]

Unhelpful error message

[![Success!](/blog/better-pc-cooling-with-python/influx2_hu9f183b9c68b128b845bbbd9f1b4f681e_171793_240x240_fit_q90_bgffffff_lanczos_3.jpg)][65]

Success!

I set the URL to `http://influxdb:8086/`, and attempted to enter credentials. I didn’t see any option to add an API key (which seems like the logical thing to connect 2 services, and is defined in `docker-compose.yml`).

Here’s where things don’t make sense. I tried the username and password in the `InfluxDB Details` section, and also the `Basic auth` section to no avail. I was greeted with `InfluxDB returned error: error reading influxDB`. This error message doesn’t help, and the logs from Grafana/InfluxDB reveal nothing too.

In the end, after reading a random forum post somewhere, I learnt that the answer is to put the _API key_ in the password section of `InfluxDB Details`. The `User` field can be any string.

FYI: The `Database` field actually means `bucket` in Influx terminology. Really, it feels like an abstraction layer that doesn’t quite fit.

### Terminology

InfluxDB has its own volcabulary. I found it a bit confusing. After reading [this thread][66], viewing [this video][67] and speaking to a friend I have this understanding when compared to a relational database:

1.  `tags` are for **invariant metadata.** They are like **columns**, and are **indexed**
2.  `fields` are for **variable data.** They are also like **columns**, but are **not indexed**
3.  a `measurement` is akin to a **table**
4.  a `point` is equivalent to a **row** (but with no schema)
5.  a `bucket` is equivalent to a **database**

It [seems to be good practice][68] including multiple reading types in a single point, so long as the data is coherent. For instance, a weather station may report wind speed, temperature and humidity on the same data point if sampled together. As far as I can see, you could also report some reading separately with no penalty to reflect the sampling used. There is no schema (“NoSQL”).

### Recording

To record, I hacked together a monitoring system based on the fan controller script that would submit readings to InfluxDB. I made it independent so a failure wouldn’t affect the control loop. The script is [here][69].

## Results

Before the new controller, you can see the fan speed ramped up and down all over the place:

[![Erratic and annoying fan speeds](/blog/better-pc-cooling-with-python/erratic_hu630d2398b5013fdeebbe9b52d5cb4d0d_63993_1280x720_fit_q90_bgffffff_lanczos_3.jpg)][70]

Erratic and annoying fan speeds

This was when building my blog (with a custom heavy optimiser) and then a [stress test.][71] Interestingly, you can see the CPU peak in temperature before setting down. The pump, set by liquid temperature by default, doesn’t spool up fast enough so the rate of cooling is less than it could be at the start – hence the peak.

With the new control scheme, the fan speed change is much more gradual:

[![Response under the new control scheme](/blog/better-pc-cooling-with-python/fangoblin3_hua4fae34a89309d6b71b509b6c76f477e_50186_1280x720_fit_q90_bgffffff_lanczos_3.jpg)][72]

Response under the new control scheme

…and that peak in CPU temperature is gone! That’s because the rate of cooling maximises immediately as the pump spools up based on CPU temperature instead of liquid temperature. The time scale is the same for both graphs.

Here’s what I had before I started experimenting:

[![bad.png](/blog/better-pc-cooling-with-python/bad_hu7432b23859070180d5efbdebf8d4a33c_44427_1280x720_fit_q90_bgffffff_lanczos_3.jpg)][73]

bad.png

In this case the pump speed was fixed. The CPU was exceeding the maximum temperature and presumably throttling as a result.

Here’s a graph during calibration:

[![fancalib.png](/blog/better-pc-cooling-with-python/fancalib_hu09877e47bda410fd564518a5c92d5a43_44212_854x480_fit_q90_bgffffff_lanczos_3.jpg)][74]

fancalib.png

Here, the script finds the minimum speed and then the maximum speed. Interestingly, this max speed is not stable – perhaps there is a curve that the fan itself applies in its firmware.

## Conclusion

Exploiting the thermal mass, and running the fans at an empirically derived minimum speed results in a significant improvement in cooling and acoustic performance.

Subjectively, the machine is now silent during idle, and doesn’t get audible unless the system is stressed for several minutes. The fans also don’t reach maximum when running games unlike before.

It is also possible to control the entire cooling stack without buying any additional control hardware, in my case.

My script above is, however, specific to my installation; so isn’t that useful outside of this post. As a result however, it’s trivial – I prefer this greatly over running a bloated GUI application.

## Future improvements

### Hybrid mode

I think a “hybrid” mode would be great. My PSU, a Corsair SF750 platinum, has a hybrid mode. In this mode (the default) the PSU operates passively (zero RPM fan) until some threshold when the fan kicks in. As a result it’s silent, but to me more importantly it’s completely spotless after 3 years in 24/7 use! No dust whatsoever.

I experimented with this by letting the system idle with the radiator fans off but the pump at 100%:

[![The liquid temperature with the fans off at idle](/blog/better-pc-cooling-with-python/nofans_hu81fca9cd2bb299336e36799b29d15b2e_212274_854x480_fit_q90_bgffffff_lanczos_3.jpg)][75]

The liquid temperature with the fans off at idle

The liquid temperature quickly approaches the recommended maximum of 60c. This tells me it probably isn’t possible without a larger radiator. I intend to investigate more thoroughly though.

This also tells me that there’s a stark difference between cooling performance at minimum (silent!) fan speed and fans off. This could result in a hunting behaviour if the control algorithm isn’t right. The system would have to leave large gaps between the passive mode being activated to avoid sporadic system use resulting in toggling between maximum fan speed and zero.

In addition, we fill up the thermal mass in the process, meaning the CPU is likely to overheat immediately if loaded in this mode before the fans kick in and the liquid temperature drop. A solution to this may be to detect if the computer is in use (mouse movements) and only allow passive mode if not. The fans would start and bring down the liquid temperature as soon as the machine is used.

### Abstraction

Making the script useful for other machines could involve abstracting coolers and sensors. The control loop for each pair could also be in a separate thread to prevent a crash in one causing the others to stop too.

### Integrated monitoring

The script could record straight to InfluxDB. This could be useful for long term analysis and assessing the impact of changing system properties – a new thermal interface compound, new fans etc.

### Stall speed detection

I mentioned earlier that the start speed of a given fan is greater than the stall speed. Providing there’s a start/restart mechanism, it should be possible to run the fans _even slower,_ resulting in even less noise and dust.

### Beat frequencies

The fans (2x radiator, 1x case) sometimes make a throbbing noise. This is due to a [beat frequency][76] being emitted when the fans are close in speed.

It’s slightly annoying. The system could drift intentionally to allow a large enough gap in rotational speed to avoid this.

### General undervolting

I’ve played with PBO2 adjustment as I said, but it should be possible to [reduce the voltage at the expense of a bit of performance.][77]

### Better fans and thermal interface compound

Finally, courtesy of a friend I have a pair of Phanteks T30s to try. Also, I have some Noctua NT-H1. They could help!

---

1.  Fluctuations are more annoying than louder fans, in my opinion. [↩︎][78] [↩︎][79]
    
2.  Likely because they have many accurate on-die sensors, with algorithms that react quickly to control power consumption before risking damage. [↩︎][80]
    
3.  See, interview code challenges _are_ relevant and useful in the real world! [↩︎][81]
    
4.  I have the luxury of having to support only one computer. No need to generalise this for other machines – though it’s easy to adapt for your purposes. [↩︎][82]
    
5.  There are many integrations, cool! [↩︎][83]
    

---

Thanks for reading! If you have comments or like this article, please post or upvote it on [Hacker news][84], [Twitter][85], [Hackaday][86], [Lobste.rs][87], [Reddit][88] and/or [LinkedIn.][89]

Please [email me][90] with any corrections or feedback.

  
[← more articles][91]

Copyright © [Callan Bryant][92] -- All rights reserved. Only [strictly necessary cookies][93] are in use.

[1]: /
[2]: /
[3]: /about/
[4]: /etc/
[5]: #the-idea
[6]: #goals
[7]: #research
[8]: #liquidctl
[9]: #lm-sensors
[10]: #sysfshwmon
[11]: #the-solution
[12]: #calibration
[13]: #the-control-loop
[14]: #installation
[15]: #measuring-performance-with-grafana
[16]: #installation--setup
[17]: #terminology
[18]: #recording
[19]: #results
[20]: #conclusion
[21]: #future-improvements
[22]: #hybrid-mode
[23]: #abstraction
[24]: #integrated-monitoring
[25]: #stall-speed-detection
[26]: #beat-frequencies
[27]: #general-undervolting
[28]: #better-fans-and-thermal-interface-compound
[29]: https://www.cpubenchmark.net/compare/3485vs3862/AMD-Ryzen-7-3700X-vs-AMD-Ryzen-9-5950X
[30]: https://nzxt.com/en-GB/product/kraken-x53
[31]: #fn:1
[32]: /blog/better-pc-cooling-with-python/IMG_8253.JPEG
[33]: /blog/better-pc-cooling-with-python/IMG_8255.JPEG
[34]: /blog/better-pc-cooling-with-python/IMG_8256.JPEG
[35]: #fn:2
[36]: /blog/better-pc-cooling-with-python/dust.jpeg
[37]: https://www.youtube.com/watch?v=dfkrp25dpQ0
[38]: https://www.youtube.com/watch?v=xmQV1ZM_WmU
[39]: https://wiki.archlinux.org/title/systemd#Writing_unit_files
[40]: https://github.com/liquidctl/liquidctl
[41]: https://www.corsair.com/uk/en/p/custom-liquid-cooling/cl-9011110-ww/icue-commander-pro-smart-rgb-lighting-and-fan-speed-controller-cl-9011110-ww
[42]: https://wiki.archlinux.org/title/lm_sensors
[43]: /blog/better-pc-cooling-with-python/sensors.png
[44]: #fn:3
[45]: https://github.com/lm-sensors/lm-sensors/blob/master/prog/pwm/fancontrol
[46]: https://en.wikipedia.org/wiki/Sysfs
[47]: /blog/better-pc-cooling-with-python/hwmon.png
[48]: https://www.kernel.org/doc/Documentation/hwmon/sysfs-interface
[49]: https://www.kernel.org/doc/Documentation/hwmon/nct6775
[50]: #fn:4
[51]: ./fangoblin3.py
[52]: javascript:;
[53]: https://en.wikipedia.org/wiki/Negative_feedback
[54]: /blog/better-pc-cooling-with-python/curve.png
[55]: #fn:1
[56]: /blog/better-pc-cooling-with-python/running.png
[57]: javascript:;
[58]: javascript:;
[59]: https://grafana.com/
[60]: https://www.influxdata.com/
[61]: /blog/better-pc-cooling-with-python/docker-compose.yml
[62]: javascript:;
[63]: #fn:5
[64]: /blog/better-pc-cooling-with-python/influx1.png
[65]: /blog/better-pc-cooling-with-python/influx2.png
[66]: https://dba.stackexchange.com/questions/163292/understanding-how-to-choose-between-fields-and-tags-in-influxdb
[67]: https://www.youtube.com/watch?v=1Iw_0J5UkYs
[68]: https://www.youtube.com/watch?v=1Iw_0J5UkYs
[69]: ./monitor-hack.py
[70]: /blog/better-pc-cooling-with-python/erratic.png
[71]: https://wiki.ubuntu.com/Kernel/Reference/stress-ng
[72]: /blog/better-pc-cooling-with-python/fangoblin3.png
[73]: /blog/better-pc-cooling-with-python/bad.png
[74]: /blog/better-pc-cooling-with-python/fancalib.png
[75]: /blog/better-pc-cooling-with-python/nofans.png
[76]: https://en.wikipedia.org/wiki/Beat_(acoustics)
[77]: https://www.youtube.com/watch?v=xmQV1ZM_WmU
[78]: #fnref:1
[79]: #fnref1:1
[80]: #fnref:2
[81]: #fnref:3
[82]: #fnref:4
[83]: #fnref:5
[84]: https://news.ycombinator.com/
[85]: https://twitter.com/
[86]: https://hackaday.com/
[87]: https://lobste.rs
[88]: https://reddit.com/
[89]: https://linkedin.com/
[90]: /cdn-cgi/l/email-protection#3655575a5a57581854444f57584276515b575f5a1855595b
[91]: ../
[92]: /about
[93]: https://support.cloudflare.com/hc/en-us/articles/200170156-Understanding-the-Cloudflare-Cookies