# TradracK1

I managed to adapt [Tradrack](https://github.com/Annex-Engineering/TradRack) to my vanilla [Creality K1](https://www.creality.com/products/creality-k1-3d-printer). It wasn't easy, so I need to write it down in case I need to do it again.

**DO AT YOUR OWN RISK**


## Preparation

Pre-requisites that I already had in place because are good ideas:
1. [Guilouz/Creality-Helper-Script](https://github.com/Guilouz/Creality-Helper-Script). Hash b46787a
2. Use the helper to install moonraker, nginx, fluidd, entware.
3. Use entware to install bash: `opkg install bash`.

## DXC Phaetus

It's a worthwhile upgrade on its own. But the main point was the filament sensor in the extruder. The installation is straightforward, but the motor will hit the chain if it's in its original position. I have a [riser](https://www.printables.com/model/520207-jc-creality-k1-lid-riser), so I just flipped it around:

<img width="1246" height="755" alt="image" src="https://github.com/user-attachments/assets/de1a4c2f-9b76-43d4-bbc5-882fd16d6172" />

The "Retract" default macro requires adjustments after this, as it doesn't fully retract the filament, and the e-steps may need recalibration. It's still relatively easy to upgrade.

## Triangle Labs Tradrack kit

### Hardware assembly

I followed the guide from their [Annex-Engineering](https://github.com/Annex-Engineering/TradRack/blob/main/docs/build_instructions/TradRack_Build_Instructions_Beta_A0.1.pdf). It's a C-type cart. Three notes:

1. Mind the side of the nuts, otherwise the next steps might not fit:
<img width="753" height="540" alt="image" src="https://github.com/user-attachments/assets/b3d44012-182f-45f7-bb3f-2557530b4df7" />

3. On the support for the selector motor, if you tighten the T bolts, you can't tighten the other screws, so mind the order:
<img width="464" height="598" alt="image" src="https://github.com/user-attachments/assets/4973ea22-669d-4246-8c5c-0ef02a60b9d0" />

4. When putting the cart in the rails, I couldn't fit the screws in following the instructions, so I put the screws in and then tightened them *carefully*:
<img width="580" height="636" alt="image" src="https://github.com/user-attachments/assets/1894b79c-2ce5-47c2-a516-4dd94c44fafc" />

To determine what to print, I followed the guide, using Overture ASA. Each part took 20-30 minutes to print; it was roughly the amount of time to assemble the previous step. I have the encoder, but didn't bother doing binky.

One relevant bit I learned after finishing building: **do not put the servo motor**. Read the [servo calibration guide](https://github.com/Annex-Engineering/TradRack/blob/main/docs/Quick_Start.md#servo-calibration) and spare yourself some headache.

### Electronics:

1. Follow the [wiring guide](https://github.com/Annex-Engineering/TradRack/blob/main/docs/Wiring.md). The underside of the board has the pinouts printed.  Need to match end-to-end.
2. The switches will be between GND and signal, not 5V.
3. Mind the jumpers, they allow you to choose which pins show up where *to some degree*. Mine came closed in the outer ones, and that's what I left, the DIAG pin of one of the TMCs is what's not connected, which means no stallguard (aka detection of the motor struggling), but I have filament switches instead (the DXC and the gate).
4. I have hooked it to the +12v/-12v of the printer power source. Can it handle it? I'm hoping so. If your printer explodes, that's on you. Mine is the white wire
<img width="1350" height="1210" alt="image" src="https://github.com/user-attachments/assets/eebce987-0961-4a26-bab0-c5febc6d7c59" />

6. Microfit terminals **are a pain to do**. I tried soldering, but then they get too thick, so I just press-fitted. A bunch came loose during troubleshooting.
<img width="722" height="512" alt="image" src="https://github.com/user-attachments/assets/f199ed78-d466-46a2-bf03-1ae66999c035" />

### Mounting:

1. I made a support that lets you hook the [tradrack in the printer](https://www.printables.com/model/1424644-tradrack-mount-for-creality-k1).
2. For the circuit board, I printed [a support in PLA](https://www.printables.com/model/481199-ercf-easybrdcan-brd-mount-for-tr).

### ERCF Easy BRD flashing

**DO NOT USE THE LATEST VERSION OF KLIPPER**. It will fail the communication. For my vanilla K1, it was version [0.12.0 from 2023](https://github.com/Klipper3d/klipper/releases/tag/v0.12.0). Just the vanilla guide. The triangle labs kit came with a Seeed Studio XIAO SAMD21, so pick that one. No other custom configuration required, no bound rate tweak or anything, just `make menuconfig` and flash it. [See this on how to enter the bootloader].(https://wiki.seeedstudio.com/Seeeduino-XIAO/#enter-bootloader-mode).

### Happy Hare

The installer needs bash, and even then, it doesn't really work in the K1. Clone the Happy Hare repo, but replace the installer with [this file](https://github.com/k1-801/Happy-Hare/blob/k1/install.sh). Adjust the version to match whatever happy hare you have and run. It will require all the parameters, like this:

``` shell
bash ./install.sh -k /usr/share/klipper -c /usr/data/printer_data/config -m /usr/data/moonraker/moonraker -i
```

Answer the questions and it should be _mostly_ fine. Two little caveats caused by Happy Hare assuming a newer version of Klipper:

1. It relies on a 'led.py` file that doesn't exist, so it will fail with a "cannot import led from extras".  Solution? Copy [the led file from the git](https://github.com/Klipper3d/klipper/blob/master/klippy/extras/led.py) into the /usr/share/klipper/klippy/extras folder.
2. If you get a cryptic error "'void(*)(struct trapq *, double)' expects 2 arguments, got 3", it's because this [commit here](https://github.com/Klipper3d/klipper/commit/d7f6348ae6e45e4b566d10974b10ab4bb111222b#diff-23b1c216e99900b3decb4e0f61af0c7c7f36a70603ff34177ab0fe069f29f492) changed the signature of `trapq_finalize_moves` from requiring 2 parameters to requiring 3. So Happy Hare passes 3, but the compiled code of Klipper only accepts two. To solve it, edit the file `/usr/share/klipper/klippy/extras/mmu_machine.py` ([Here's the exact line on git](https://github.com/moggieuk/Happy-Hare/blob/ef044c4b094151c69a86cce0c338bac4fdf177e9/extras/mmu_machine.py#L692)) to:
   ```python
   def _finalize_if_valid(tq, t):
    if tq is not None and tq != ffi_main.NULL:
        self.trapq_finalize_moves(tq, t) #, t - MOVE_HISTORY_EXPIRE)
   ```

### Servo calibration

Follow the [servo calibration guide](https://github.com/Annex-Engineering/TradRack/blob/main/docs/Quick_Start.md#servo-calibration). However, before you start, set the position to 0 and then to the max it goes, usually < 180 (if you put 10000, it will go to max), and see how much it actually turns. From "factory" mine was doing only 90 degrees, and that's not nearly enough to fully lift the assembly.

That's because the 1171MG angle is controlled by pwm, the "center" is represented as 1500ms, do a pulse with a lower ms and it goes to one way, higher goes the other. The limits on the configuration are causing those 90 degrees. Change the min and the max pulse widths to these values and then continue the same guide.

``` cfg
[mmu_servo selector_servo]
pin: mmu:MMU_SEL_SERVO
maximum_servo_angle: 180
minimum_pulse_width: 0.0005 
maximum_pulse_width: 0.0025
```

### Carry on with calibration

Those were all the little caveats I found. Otherwise, just follow the guides: 

https://github.com/moggieuk/Happy-Hare/wiki/Hardware-Configuration

I also put my config files in this git for reference. I still have some calibration and kinks to work out (it's doing filament change without parking the head, and it's trying to home the Z axis on MMU_END), but the basic tool change during print is working.

# References:

* https://www.instructables.com/ERCF-V2-for-Creality-K1K1MaxK1C/
* https://sakitume.github.io/Custom-ERCF-Build-Sakitume-Edition/sidebar/happy-hare.html
* https://www.teamfdm.com/forums/topic/2307-tradrack-another-mmu/
* https://wiki.seeedstudio.com/Seeeduino-XIAO/
