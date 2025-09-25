# TradracK1

I managed to adapt [Tradrack](https://github.com/Annex-Engineering/TradRack) to my vanilla [Creality K1](https://www.creality.com/products/creality-k1-3d-printer). It wasn't easy, so I need to write it down in case I need to do it again.


## DXC Phaetus

It's a worthwhile upgrade on its own. But the main point was the filament sensor in the extruder. The installation is straightforward, but the motor will hit the chain if it's in its original position. I have a [riser](https://www.printables.com/model/520207-jc-creality-k1-lid-riser), so I just flipped it around:

<img width="1246" height="755" alt="image" src="https://github.com/user-attachments/assets/de1a4c2f-9b76-43d4-bbc5-882fd16d6172" />

The "Retract" default macro requires adjustments after this, as it doesn't fully retract the filament, and the e-steps may need recalibration. It's still relatively easy to upgrade.

## Triangle Labs Tradrack kit

### Hardware assembly

I followed the guide from their [Annex-Engineering](https://github.com/Annex-Engineering/TradRack/blob/main/docs/build_instructions/TradRack_Build_Instructions_Beta_A0.1.pdf), with only 3 pain points:

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
4. Microfit terminals **are a pain to do**. I tried soldering, but then they get too thick, so I just press-fitted. A bunch came loose during troubleshooting.
