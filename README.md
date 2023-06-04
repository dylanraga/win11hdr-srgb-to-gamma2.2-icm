# Transform Windows 11's virtual SDR-in-HDR curve from piecewise sRGB to Gamma 2.2

## <a href='#icm-dl'>Jump to downloads</a>

Windows 11 has made strides in streamlining the HDR experience, finally allowing for casual workstations where HDR is always enabled throughout the desktop. This is made possible by emulating an SDR color space within the native HDR10 composition space so that both types of content can coexist.

However, the specifications for "SDR" can sometimes be ambiguous since there are multiple standards that depend on viewing parameters.

The rule of thumb is:

- Gamma 2.4 — _Rec.709 reference. Dark room viewing (5 lux). Home theater use._
- Gamma 2.2 — _Pseudo-standard in uncontrolled lighting (>50 lux). Computer use._

As of version 22H2, Windows 11 does not allow us to pick which tone curve to use for SDR — in fact it uses _neither_ of the above. Instead, Windows 11 uses what's called the _piecewise sRGB transfer function_, originally intended for computer/web use. This curve is technically the tone response that gamma 2.2 approximates, but for various reasons it wasn't commonly used as an actual display EOTF. Gamma 2.2 took over as the pseudo-standard, and almost every computer monitor still uses it to this day.

Because it's so prolific, almost all web content and computer games are color graded using a gamma-2.2 outputting monitor, intentionally or not. For Microsoft to decide using piecewise sRGB as the virtual SDR curve was a mistake; it forcibly renders almost all apps and web content to look incorrect, some more than others.

As an example, we can take a look at Diablo IV, a game that is meant to be dark and gritty.

| ![Diablo IV in SDR (gamma 2.2 tone curve)](./d4_gamma2p2.png) |
| ------------------------------------------------------------- |
| _Diablo IV in SDR (gamma 2.2 tone curve)_                     |

| ![Diablo IV in HDR (sRGB tone curve)](./d4_srgb.png) |
| ---------------------------------------------------- |
| _Diablo IV in HDR (sRGB tone curve)_                 |

Diablo IV looks absolutely spectacular in SDR, with deep shadow detail and excellent use of dithering to prevent blacks from appearing blotchy.

On the other hand, there have been countless complaints about the game looking washed out when HDR is enabled, swaying many players to simply enjoy the game in SDR. When HDR is enabled, rich blacks turn into dull grays, and the game's atmosphere turns hazy, losing much of its depth. The primary cause of this is — _you guessed it_ — the piecewise sRGB tone curve that Windows uses in HDR.

<em><small>Note: Only games with HDR implementations that output to scRGB are susceptible to this issue, which includes any game enhanced by Window's Auto-HDR. Engines that output to ST2084 are not affected.</small></em>

We can plot the output luminance of sRGB vs. gamma 2.2 to see the the concrete differences:

| ![Gamma 2.2 vs Piecewise sRGB chart](./srgb_vs_g22.png)                                                                                   |
| ----------------------------------------------------------------------------------------------------------------------------------------- |
| _Gamma 2.2 vs Piecewise sRGB chart. The luminance axis is in log space since our eyes perceive the sensation of lightess logarithmically_ |

Here, the discrepancy between gamma 2.2 and sRGB is obvious. The sRGB curve is fundamentally much lighter below the mid-tones, making content mastered in gamma-2.2 appear flat when viewed with an sRGB transfer.

macOS currently offers a nice solution, allowing its users to create their own custom reference mode in the dispay settings. Among a few others, one of the custom parameters is the _SDR Transfer Function_, which the user can pick between BT.1886 (for film), a pure gamma (2.2, for PC use), or piecewise sRGB (for treason).

| ![macOS custom reference mode](./macos_crf.png)                               |
| ----------------------------------------------------------------------------- |
| _macOS custom reference mode allows the SDR Transfer Function to be adjusted_ |

# Temporary solution

This is a problem that ultimately requires Microsoft to fully rectify, so in the meantime we'll need to resort to a band-aid fix. By using an MHC2 ICC profile, we can apply a system gamma ramp that transforms the SDR sRGB curve into a 2.2 gamma curve.

The caveat with this solution is that _all_ content takes on the same transformation, including native HDR10 implementations, which don't need it; such content will see slightly darker shadows, potentially crushing shadow detail. You can either disable the color profile when viewing native HDR content, or just live with the slight boost in contrast for the sake of convenience.

An accurate curve mapping does depend on Window's _SDR content brightness_ value, so I've provided several profiles to choose from — pick the one closest to your setting. The transformation is also an _ideal_ mapping, so its accuracy will also depend on your monitor and its calibration quality.

<h2 id='icm-dl'>Download srgb_to_gamma2p2 color profile</h2>

- [100 nits / Brightness 5](./srgb_to_gamma2p2_100_mhc2.icm)
- [200 nits / Brightness 30](./srgb_to_gamma2p2_200_mhc2.icm)
- [300 nits / Brightness 55](./srgb_to_gamma2p2_300_mhc2.icm)
- [400 nits / Brightness 80](./srgb_to_gamma2p2_400_mhc2.icm)

## Windows SDR content brightness table

| SDR brightness value | SDR white screen luminance |
| -------------------- | -------------------------- |
| 0                    | 80 nits                    |
| 5                    | 100 nits                   |
| 10                   | 120 nits                   |
| 30                   | 200 nits                   |
| 55                   | 300 nits                   |
| 80                   | 400 nits                   |
| 100                  | 480 nits                   |

## Instructions

1. Download one of the color profiles above
2. Open Windows' "Color Management" utility
3. Press "Add" on the bottom left
4. Press "Browse..." and select the color profile you downloaded
5. Enable the "Add as HDR Profile" checkbox
6. Press "OK"
7. Select the color profile you added and press "Set as Default Profile"
8. Done

## Notes

- All profiles are tagged with a peak luminance of 800 nits and a black level of 0
- Pixel values above diffuse SDR white are untouched; a soft shoulder was added toward unity to blend the curve mapping with HDR values
- Tested on an LG C2 OLED

<hr>

<small><em>These color profiles were created with the help of [MHC2Gen](https://github.com/dantmnf/MHC2/tree/master/MHC2Gen) and [ArgyllCMS](https://www.argyllcms.com/).</em></small>
