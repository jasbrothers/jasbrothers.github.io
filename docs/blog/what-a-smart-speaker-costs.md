# What a Smart Speaker Actually Costs

<p style="text-align:center;margin:1.6em 0"><img src="../images/speaker-solid.svg" alt="The Jas Bros smart speaker: a printed enclosure with a speaker grille, the JAS BROS wordmark, side vents and two microphone ports in the top" style="max-width:380px;width:100%;height:auto"></p>

One hundred and thirteen dollars.

That is every part in the box — the computer, the microphone array, the speaker,
the case, the screws, the foam, the box the parts ship in. Below is the whole
list, and then the arithmetic that turns it into a shelf price.

We are publishing this because we are about to ask people for money, and the
first honest thing you can do in that situation is show your costs.

## The box the parts go in

Every part on the list has to physically fit somewhere, so we drew the enclosure
before we trusted the prices. It is two printed parts — a body shell and a base
plate — and everything else bolts into them.

<p style="text-align:center;margin:1.6em 0"><img src="../images/speaker-exploded.svg" alt="Exploded view: base plate, Raspberry Pi 4, body shell, 5W driver, acoustic foam and the reSpeaker Lite, separated along the assembly axis and labelled" style="max-width:820px;width:100%;height:auto"></p>

Bottom to top: the base plate carries the Pi on four standoffs, the driver bolts
to the inside of the front baffle behind the grille, a foam pad damps the rear
wall, and the reSpeaker Lite hangs from the ceiling so its two microphones sit
directly under the two ports in the top face.

<p style="text-align:center;margin:1.6em 0"><img src="../images/speaker-assembled.svg" alt="The same speaker with the shell drawn transparent, showing how the components stack inside" style="max-width:400px;width:100%;height:auto"></p>

**110 × 92 × 152 mm**, 2.8 mm walls. About **225 g** of filament for the shell
and **51 g** for the base plate. Both parts sit on a single Bambu P1S plate with
room to spare — 230 × 92 mm of a 256 × 256 bed.

Three decisions in there are printability, not styling:

- **The grille is sixty-one 4 mm holes, not one 52 mm opening.** A big circular
  hole in a vertical wall overhangs badly at the top of the arc. A hole array
  does the same acoustic job and every hole is small enough to bridge.
- **The shell prints upside down** — top face on the bed, open end up. Printed
  the other way, that top face would have to bridge the whole 104 × 86 mm
  cavity, which no printer will do cleanly.
- **The rear I/O opening runs off the bottom edge** rather than being a closed
  window. As a notch it needs no bridge at all, and the base plate closes it
  from below anyway.

The files, if you want to print one:

- **[smart-speaker-enclosure.3mf](files/smart-speaker-enclosure.3mf)** — both
  parts, print-oriented, on one plate
- **[body shell STL](files/smart-speaker-body-shell.stl)** ·
  **[base plate STL](files/smart-speaker-base-plate.stl)**

Fair warning: this is drawn, checked and exported, but **it has not been sliced
or printed yet**, and no part has been test-fitted in the real world. The
component clearances are verified against the published dimensions, not against
parts on a bench. When we print one, whatever is wrong with it goes in a
follow-up post.

## The list

| Category | Component | Qty | Unit | Ext. |
|---|---|---:|---:|---:|
| Compute | Raspberry Pi 4 Model B (2GB) | 1 | $45.00 | $45.00 |
| Compute | microSD card 16GB (A1) | 1 | $5.50 | $5.50 |
| Compute | USB-C PSU (5V/3A) | 1 | $6.00 | $6.00 |
| Compute | Passive heatsink set | 1 | $2.00 | $2.00 |
| Audio capture | reSpeaker Lite (XMOS XU316) | 1 | $24.90 | $24.90 |
| Audio capture | USB-C cable (short) | 1 | $3.50 | $3.50 |
| Audio output | 5W full-range driver, 4Ω | 1 | $7.00 | $7.00 |
| Audio output | JST-PH 2-pin speaker pigtail | 1 | $0.75 | $0.75 |
| Cables | Board mounting hardware | 1 | $2.00 | $2.00 |
| Cables | Internal wiring kit | 1 | $1.50 | $1.50 |
| Enclosure | 3D-printed shell | 1 | $9.00 | $9.00 |
| Enclosure | Acoustic damping foam | 1 | $2.00 | $2.00 |
| Enclosure | M2.5 fastener set | 1 | $1.00 | $1.00 |
| Packaging | Kit box + foam insert | 1 | $2.50 | $2.50 |
| Packaging | Quick-start card | 1 | $0.35 | $0.35 |
| **Total** | | | | **$113.00** |

Single-unit pricing, August 2026, from Seeed and the usual distributors. At a
thousand units most of these fall 25–40%, especially the Pi. We are not counting
on that yet.

## The microphone array

The cheapest way to build this is two bare microphones wired straight to the Pi.
We are not doing that, and it is worth being precise about why.

The box talks while it is listening. You say "Hey Claude," it starts answering,
and you want to be able to interrupt it — which means the microphones are live
while the speaker eighteen inches away is playing. Without **acoustic echo
cancellation**, the box hears itself, and everything downstream falls apart: the
wake word fires on its own voice, and the transcription is a transcript of the
answer it just gave.

AEC is not a nice-to-have on a device shaped like this one. It is the thing that
makes it a speaker instead of a walkie-talkie. You can do it in software, on the
Pi, competing for cycles with Whisper — or you can buy a board with a chip that
does it and stop worrying. For fifteen dollars over a bare codec, we buy the
chip.

Here is the whole shelf, since we priced all of it:

| Board | Price | Hardware DSP | Mics / range |
|---|---:|---|---|
| **reSpeaker Lite (XU316)** | **$24.90** | AEC, interference cancellation, noise suppression, AGC. **No beamforming.** | 2 mics, 3 m |
| reSpeaker XVF3800 (no XIAO) | ~$50–61 | AEC, **multi-beamforming**, de-reverberation, DoA, NS, 60 dB AGC | 4 mics, 5 m |
| reSpeaker Mic Array v2.0 (XVF3000) | $64.00 | AEC, beamforming, DoA, NS | 4 mics, 5 m |
| reSpeaker 4-Mic Array HAT (AC108) | $24.90 | **None.** It is a codec. | 4 mics |

**We picked the reSpeaker Lite, at $24.90.** It keeps hardware AEC, noise
suppression and AGC, and it carries a speaker connector rated for 5W plus a
3.5mm output. That makes it the microphone array *and* the digital-to-analog
converter *and* the amplifier, which is why there is no separate DAC or amp
anywhere on the list.

What it does not do is beamforming. The four-mic boards can steer toward whoever
is talking and tell you what direction they are in; the Lite has two microphones
and cannot. Its far-field spec is 3 metres against 5. For a box on a shelf in a
normal room, we think 3 metres and no beam-steering is enough. **We think.** In a
kitchen with a dishwasher running, it might not be, and if the room test says so
the answer is the XVF3800 and the kit price goes from $249 to $309.

## Three more decisions

**A 2GB Pi, not a 4GB one.** Whisper runs on this machine, locally, and the model
has to fit in memory alongside the operating system. `base.en`, quantized, should
sit inside 2GB. Should. If it does not, it is a 4GB board and ten dollars more.

**One driver, not two.** It is a box that answers questions, so there is nothing
to put in a second channel. A single 5W full-range driver, run straight off the
array board's speaker connector — no crossover, no second amplifier, no stereo
image nobody would hear anyway.

**No fan.** A fan two inches from a microphone is not a cost saving, it is a bug.
Passive heatsink, no moving parts, nothing for the array to have to cancel.

And it ships as a **kit** — parts, a printed shell, instructions. No assembly
labour, no retail packaging, no sealed box with a warranty sticker. That is worth
about $34 a unit, and it suits the person who wants this thing better than a
sealed box would.

## The part we refuse to cut

You could build this for forty dollars.

Take out the Pi 4, put in a Pi Zero 2 W. Stop running speech recognition on the
device and stream the microphone audio to a server instead. Now you need almost
no memory, almost no processor, no heatsink. Forty dollars, maybe less.

You would also be shipping a microphone that streams your living room to somebody
else's computer. That is the thing this box exists not to be. The wake word runs
here. Whisper runs here. Exactly one step in the loop — asking the question —
leaves the house, and only after you have said the words out loud on purpose.

**The privacy promise has a bill of materials.** It is the difference between a
$45 computer and a $15 one, and it is most of the reason this costs what it
costs. We would rather explain that number than not have to.

## Turning $113.00 into a price

Parts are not cost. Getting one kit into one box also takes about **$8.00** of
somebody counting parts and packing them. So a kit costs us **$121.00**.

We want a 50% gross margin — a plain way of saying the price is twice what the
thing costs us:

| | Kit | Assembled |
|---|---:|---:|
| BOM | $113.00 | $113.00 |
| Finished shell + retail box | — | $9.00 |
| Assembly, flash, bench test | — | $25.00 |
| Pick & pack | $8.00 | $8.00 |
| **Cost per unit** | **$121.00** | **$155.00** |
| **Price** | **$249** | **$319** |
| **Gross margin** | **51.4%** | **51.4%** |

Early backers get the kit at **$219**, a 44.7% margin. That tier is thinner on
purpose. It is what going first is worth.

## What 50% does not mean

Fifty percent gross margin is not fifty cents of every dollar in our pocket, and
anyone who tells you otherwise is selling something. Out of the $128.00 of gross
profit on a $249 kit:

- **Kickstarter and payment processing take about 8.5%** — roughly $21.17.
- **A failure reserve of 5%** — about $12.45. Units die in the post. Boards
  arrive dead. People need help. Not budgeting for that is how a campaign turns
  into a year of unpaid support work.
- **Shipping is collected separately** at pledge time and passed straight
  through. It is not margin and we will not pretend it is.

What is left is **$94.38 a kit, about 37.9%**. That is the number that says
whether this survives contact with reality, and it is the number we watch.

**What is not in any of this:** certification, tooling, and our own time. FCC and
CE testing is a real cost for anything with a radio in it and we have not priced
it yet — when we do, it goes in this table like everything else. Nobody is paying
themselves either.

At $12,000 of fixed costs — first parts buy, print tooling, test gear,
photography — and a mix of roughly 70% kits, break-even is about **118 units**, or
a little under $32,000 in pledges.

## The spreadsheet

Here it is. Both tabs, every formula live. Change a quantity or a unit cost and
the price, the margins and the break-even all move with it. There is a **mic
array selector** on the BOM tab: type `Lite`, `XVF3800`, `v2.0` or `HAT` into one
orange cell and the whole model re-prices around that board. That is the cell we
expect to argue about.

- **[smart-speaker-bom.xlsx](files/smart-speaker-bom.xlsx)** — BOM and pricing model
- **[smart-speaker-bom.csv](files/smart-speaker-bom.csv)** — just the parts list

If a number in here is wrong, we would genuinely like to know. Several are
catalogue estimates rather than quotes, and the person who has actually bought
three hundred speaker drivers knows something we do not. Assume at least one line
is wrong and tell us which.

## Sources

- [reSpeaker Lite — Seeed Studio](https://www.seeedstudio.com/ReSpeaker-Lite-p-5928.html)
  and the [reSpeaker Lite wiki](https://wiki.seeedstudio.com/reSpeaker_usb_v3/)
- [reSpeaker XVF3800 USB Mic Array — Seeed Studio](https://www.seeedstudio.com/ReSpeaker-XVF3800-USB-Mic-Array-p-6488.html)
  and the [XVF3800 wiki](https://wiki.seeedstudio.com/respeaker_xvf3800_introduction/)
- [reSpeaker Mic Array v2.0 — Seeed Studio](https://www.seeedstudio.com/ReSpeaker-Mic-Array-v2-0.html)
- [reSpeaker 4-Mic Array for Raspberry Pi — Seeed Studio](https://www.seeedstudio.com/ReSpeaker-4-Mic-Array-for-Raspberry-Pi-p-2941.html)

<!-- TODO: add Kickstarter prelaunch "Notify me" link here before publishing -->
