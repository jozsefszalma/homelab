# Home Lab

The bare metal in my basement  

## The Platform

For my home lab I bought a used [Gigabyte G292-Z20 HPC/AI Server](https://www.gigabyte.com/Enterprise/GPU-Server/G292-Z20-rev-100) through eBay in January 2024.  

This is basically a data center server intended for AI workloads; for those who regularly work with AWS, this is roughly comparable to the technology behind the [Amazon EC2 G4dn Instances](https://aws.amazon.com/ec2/instance-types/g4/).  

I purchased it without the GPUs (it can accommodate 8x double slot Nvidia T4 or V100 cards), but it is still ridiculously overpowered as-is (24 core CPU, 8 channel 512GB DDR4 ECC2 memory, two 2.2 kW power supplies, two 10GbE network connections etc).  

I assume the price was 'right' because majority of the business workloads these days are running on at least A10 or A100 GPUs, so a server that was designed for the Volta/Turning architectures is somewhat passé for enterprise deployment and who else would be crazy enough to purchase something like this right?

![G292 Z20](images/G292_Z20_official_1.png)

## Challenges

Why this is generally a bad idea and PITA.  

If your only goal is to play with large AI models at home, then as of early-2024 a Threadripper-based workstation (or perhaps a high-end M2/M3-based Mac) would be a much better, although more expensive option.  

* **Physical Dimensions:** this thing is huge (and heavy); in order to accommodate the 8 GPUs the case is over 80 cm deep, which means finding (and transporting) a cheap rack for it is not easy.  
![Warning sticker](images/two_person_lift.jpeg)  

* **Noise Levels:** Unmodified, the fans generate [noise levels](https://audiology-web.s3.amazonaws.com/migrated/NoiseChart_Poster-%208.5x11.pdf_5399b289427535.32730330.pdf) close to 100 dB during boot, certainly not suitable for residential environments.
 ![Fans in the case](images/fans_1.png)

## Let's Mod It

### The Fans

There are no words to describe how insane the [stock fans](https://www.delta-fan.com/technology/three-phase-fan/pfm0812he-01bfy.html) are. They literarily SCREAM and draw over 4 amps each at max speed...and there are 8 of them.  
![Wind tunnel gif](images/wind-windy.gif)

The first thing I did was obviously adjust the fan curves in the management console and enable the [ErP Lot 9](https://www.eceee.org/ecodesign/products/enterprise-servers/) mode in the BIOS (which, among other things, puts the 2nd power supply into cold standby at low loads). This helped getting the noise down to a bearable 60 dB at idle, but this level was still very distracting.

At first I was thinking of replacing the stock fans with some Noctua fans and call it a day, however let's compare some specs here (all 80mm 12V fans):  

| type | Static Pressure | Air Flow | Noise (dB) | min RPM | max RPM
|------|------|------|------|-----|-----|
| Delta (stock) | 126.22 mm H₂O | 3.67 m³/min | 77.0 dB | 3000 | 16300 
| Noctua NF-A8-PWM | 2.37 mm H₂O | 0.93 m³/min | 17,7 dB | 450 | 2200 

Since I don't want to gut the cooling capabilities of the system entirely, I can't go with Noctua, which are silent, but don't move much air in exchange.  

I decided to try and optimize the idle fan speed instead, since the stock fans don't go below 3000 RPM.  
  
This is how I found Arctic's 10k server fans. Arctic doesn't seem to disclose noise levels (apart from the fans being 'quiet'), however they have reasonable static pressure, air flow performance and can spin as slow as 500 RPM.  

| type | Static Pressure | Air Flow | Noise (dB) | min RPM | max RPM
|------|------|------|------|-----|-----|
| ARCTIC S8038-10K | 51 mm H₂O | 2.89 m³/min | ? | 500 | 10000

Only later did I learn that while the Arctic fans can spin as slow as 500 RPM, the warning thresholds in BIOS will reset to 1500 (non-critical) / 1200 (critical) after a reboot, which means the fan controller will try to compensate the delinquent slow spinning fans by spinning everything up to max speed temporarily.  
To compromise I set the fan curves to 15% at idle, 1650 RPM.  

Not great, not terrible.  

The next challenge was connecting the new fans as the fan header is not a standard PC header.
![Fan header](images/fan_header.png)

It looks like a 7 pin Molex connector, with 6 leads connected, while 4 wires leading to fan.  
The pinout of the original fan was available on the manufacturer's site.  

![Pinout 7 pin Molex](images/pinout.png)  
Based on this I assume the extra black and red wires are just there to ensure the connector can deliver the amperage needed by the fan.  

For reference, the pinout of the Arctic (or any normal PC fan):  
![Pinout PC fans](images/artic_pinout.jpeg)  

Since I didn't have the right 7 pin Molex connectors or the proper crimping tool (and I didn't want to cut the connectors off the original fans just yet), I worked on the rack while I waited for the connectors to arrive.  

### The Rack
The cheapest suitable rack I found that I didn't need to rent a truck to transport was StarTech's open frame server rack line. It looks deceptively timid on marketing photos, but it has excellent build quality.  

However, the fact that this was an open frame rack didn't help the noise level situation.  

![The rack](images/rack.jpg)

While waiting for the fan connectors to arrive, I decided to take a somewhat unorthodox and perhaps drastic action and glue some acoustic foam to MDF plates and hang these improvised 'acoustic panels' on two sides and the top of the rack.  

![Rack with acoustic panels](images/three_sides.png)

Both MDF and the foam have good acoustic properties, but both do accumulate static electricity and both shed some dust particles. So, yea not ideal for computers.  

The static charge seem to be dissipated by the metal rack, the rack is grounded, the server is also grounded and enclosed, so not an immediate concern.  

**The three panels (so not complete enclosure), got me approximately 10 dB decrease in noise levels**, so halved the perceived noise.

### Finally, After a Lot of Crimping

Once I crimped new connectors on the Arctic fans (for future reference, [Molex 22013077](https://www.molex.com/en-us/products/part-detail/22013077)) **the noise levels decreased to 39 dB at idle (quiet library) and 55 dB during boot (dishwasher)**; for context the ambient noise is around 34 dB in the given room.
In exchange the CPU is running approximately 10C higher temperatures during Mixtral CPU inference (75C instead of 65C), which is still acceptable.  

This is not the best possible result as

* I didn't attempt to replace the power supply fans; as per the Internet lot of people try this and fail (something about underpowered fans and the PSU slowly cooking itself)
* I haven't done anything in terms of the back covers yet; obviously most of the noise is escaping through the back...

The PSU fans tend to change pitch slightly for some reason (perhaps showing their age) and at high CPU fan RPMs the case resonates (even with rubber dampeners) so the work will need to continue...  

![eye-twitch gif](images/eye.gif)

## Power Draw & Thermals

"But Josef, what about your electricity bill?!" asked a colleague when learning about the dual 2.2 kW power supplies.  
As per the management console the system (without GPUs) is drawing between 175 to 200 watts idle, 300 to 325 watts under load.  

Training the 124M param GPT2 on CPU using [karpathy/nanoGPT](https://github.com/karpathy/nanoGPT) for 20 minutes (98% cpu util) resulted in CPU temps hovering around 76C and occasional max readings of 78C, with fan speeds reaching max 5550 RPM.

![this is fine meme, without flames](images/this-is-fine.jpg)


## Safety First

Obligatory disclaimer: in case I observe any sudden drops in loss, I will operate the AI-safety equipment illustrated below.
![Alt text](images/ai_safety.png)  


## After 2 Months

A couple of things I learned meanwhile:
* Don't be fooled by the manual, the BIOS does NOT have an option for Wake on Lan (WoL), if the board comes with the 10GbE Mellanox network adapter. I went as far as installing [Nvidia Firmware Tools (MFT)](https://network.nvidia.com/products/adapter-software/firmware-tools/) to enable WoL on the adapter, but in the end it looks like I might have just used the wrong app / wrong parameters to send the WoL packet and it might just work by default.
* The ARCTIC fans appear to be more than enough for single slot Nvidia datacenter cards (YMMV).
* If you are thinking about UPS-sizing, consider the wattage of your preferred GPUs before you panic at the prices of > 2000 VA units...you might not need a large unit.
* The setup of Ubuntu 22.04 and config for deep learning went totally uneventful, do recommend. 
* Outstanding question if one could mix various datacenter GPUs (mixed GPU population is not supported as per the manual), but as long as datacenter is not mixed with consumer hardware I don't see why not (to be tested).




