# Level 2 Programming

Tejas is six. He reads Level 2 books. Over a weekend he built a smart speaker:
you say "Hey Claude," ask a question out loud, and the laptop answers out loud.

He did not type a line of Python.

## It started on a sheet of paper

![The Claude Speaker design sketch: a person says "Hey Claude" to a box labeled Claude, and the box answers](images/claude-speaker-design-1.jpg)

Two panels, drawn in pen. On the left, a stick figure says **"Hey Claude."**
Sound waves travel across the page into a box labeled *Claude*. On the right,
panel **2**, the box talks back: **"the answer is…"**

That is the entire product. A person talks, the box answers. The word *Claude*
is spelled a slightly different way nearly every time it appears on the page,
which does not make the design any less clear.

Then page two:

![The requirements page: "Requirements: 1) use python, 2) can run on osx laptop. Questions: 1) How can it connect to spotify?"](images/claude-speaker-design-2.jpg)

**Requirements**

1. use python
2. can run on osx laptop

**Questions**

1. How can it connect to spotify?

Look at what is on those two pages. A picture of the product. Two hard
constraints on how it gets built. One open question, written down instead of
guessed at.

That is a design document. It is missing nothing important.

## What happened next

The two photos went to [Claude Code](https://claude.com/claude-code), which
first turned them into a written design — the sketch expanded into modules,
interfaces, and a build order — and then wrote the code.

That weekend it was five small Python files and a loop:

```python
while True:
    wake.wait_for_wake()                     # listen for "Hey Claude"
    audio = audio_in.record_until_silence()  # record the question
    question = stt.transcribe(audio)         # Whisper, running locally
    answer = brain.ask(question)             # the Claude API
    tts.speak(answer)                        # say it out loud
```

Both requirements held. It was Python, and it ran on the laptop with no extra
hardware — the built-in microphone and the built-in speakers were the whole
device. Only one of those five steps used the internet; the microphone, the wake
word, and the speech recognition all ran on the machine, so the room was not
streamed anywhere until somebody actually said the wake word.

It is [rather more than five files now](https://github.com/greg1232/hey-claude/tree/main/src),
and it lives on a Raspberry Pi on a shelf instead of a laptop. That loop is
still the middle of it, and so is the rule about the room.

## The wake word was the hard part

The drawing says "Hey Claude," and the drawing is the specification.

The trouble is that openWakeWord — the small local detector that listens for
the phrase — ships with only a handful of pre-trained wake words, and "hey
claude" is not one of them. The honest options were to accept "hey jarvis" or
to train a new one.

We trained a new one, on the same laptop. No GPU, no cloud, and nobody had to
say "Hey Claude" into a microphone even once: a text-to-speech engine says the
phrase thousands of times in hundreds of different synthetic voices, a pile of
unrelated speech and music and noise serves as the counterexamples, and a small
network learns to tell the two apart. About a gigabyte of downloads and an hour
of waiting.

On the synthetic voices it scored **0.99**. This post used to end the story
there.

Then we said "Hey Claude" at it.

It scored **0.001**, and woke on seven recordings out of eighty. What we had
built was an excellent detector of a text-to-speech engine saying "Hey Claude,"
which is not a thing anybody in this house does.

Recording eighty real utterances from four people took it from 9% to 80%, and
broke the other end instead: about 180 false wakes an hour on the real
microphone. Forty minutes of recording the room — television included — cut that
twelvefold. Making the trainable part three times bigger lifted the whole curve.
None of it touched the actual problem, which was that recall and false wakes
moved together at every setting. The model was sliding one dial, not telling two
things apart.

The reason is in the shape of the tool. openWakeWord is a mel filterbank, a
**frozen network from 2020** with 0.33M parameters, and a small head you train on
top. Timed on the Pi, the frozen part is 91% of the work and the part you are
allowed to change is 1%. If the frozen 91% cannot already hear the difference
between this family saying "hey Claude" and this family saying anything else,
nothing you bolt on top will recover it. And it cannot.

So we threw it away and used Whisper's encoder instead — the same speech model
that transcribes the questions, which has heard rather more speech than 0.33M
parameters can hold. Feed it two seconds of audio, keep 768 numbers, fit a
logistic regression on those. Same recordings, same room:

| | openWakeWord, best | **Whisper encoder** |
|---|---:|---:|
| Recall at ~55 false wakes/hr | 42% | **84%** |
| Recall at ~125 false wakes/hr | 59% | **91%** |
| Cost on a Pi 4 | 0.16× realtime | 0.42× realtime |

Double the recall for two and a half times the compute, on a machine with three
cores to spare. The trained part is 11 kB.

The nice consequence is that retraining is nearly free. The expensive step is
Whisper's encoder, and it has already run by the time the speaker wakes — so
every firing writes down its 768 numbers, and fitting a new model on a day of
them takes 0.67 seconds. The speaker retrains itself at four in the morning on
what it got wrong the day before, and refuses the new model unless it beats the
old one on the firings a person actually sat and listened to.

The openWakeWord model is still on Hugging Face as
[**gdiamos/hey-claude**](https://huggingface.co/gdiamos/hey-claude) if you want
it. It is the best version of the wrong approach. The
[whole story is written down](https://github.com/greg1232/hey-claude/blob/main/docs/wake-word.md)
— where to put the threshold, how it learns from being wrong, and the
second-stage verifier that looked obvious and was not.

## Level 2

Children's books have a number printed on the cover. Level 1 is a few words a
page, with the picture doing most of the work. Level 2 is short sentences and a
real story, read mostly on your own, with somebody nearby for the hard words.
Level 3 is chapters.

Tejas reads Level 2.

Now look at that requirements page again. *use python.* *can run on osx laptop.*
*How can it connect to spotify?* Short declarative sentences. One idea each.
Nothing hedged, nothing subordinate, no sentence that needs a second reading.

That is Level 2 prose. It was also a complete enough specification to build
working software from.

Programming used to start at the other end of the shelf. Not Level 2, not even
the chapter books — the manuals. You could not make your first real thing until
you could read and write the way adults write for other adults: syntax, types,
stack traces, documentation. The distance between *wanting something* and
*being able to ask for it* was measured in years, and most people who wanted
something quit somewhere in the middle of that distance.

That distance is what collapsed. Not the difficulty of building software — the
reading level required to ask for it.

Asking well is still a real skill, and this is where the sketch is genuinely
good. It does every job a specification has to do:

- **It shows the product working**, from the outside, from the user's side.
- **It names the constraints that actually constrain** — Python, macOS laptop —
  and stays quiet about the hundred decisions that did not matter.
- **It writes down what nobody knows yet** instead of hiding it. *How can it
  connect to spotify?*

That last one is the most grown-up thing on the page. A question you have
written down is a question somebody else can answer. The Spotify answer turned
out to be tools: Claude does not play music itself, it asks our code to, and our
code calls Spotify. It went into the design document in full — and then it got
deferred, because getting the speaker talking was worth more than getting it
singing. Knowing which one to build first is a Level 2 skill too.

None of this says the hard part went away. Somebody still had to notice that the
wake word did not exist and go make one, and that was not a Level 2 job. But it
is no longer the *first* job. The first job is knowing what you want and being
able to say it in short sentences.

The drawing was the source code. Everything else was reading the hard words out
loud.

## What it does now

This post used to end with a list of what to build next: play music by voice —
the Spotify question, finally answered out loud — then weather, timers, web
search, a light that shows when it is listening, remembering conversations
between runs, and moving the whole thing off the laptop onto a Raspberry Pi so
it is a real box sitting on a shelf, which is what the drawing showed all along.

All of it is done except the remembering.

It plays Spotify, gives the forecast, sets timers and alarms that survive a
reboot, searches the web when the answer turns on something recent, reads books
aloud from a shelf of 48,284 and picks up where it left off the next evening,
makes rain and ocean and a fireplace out of filtered noise rather than looped
recordings, and can look up what a bullfrog actually sounds like. A ring of
twelve lights goes blue while it listens and green while it talks. It runs on a
Pi on a shelf, comes back on its own after a power cut, learns a new voice from
somebody repeating the wake word at it, and writes down the things it was asked
for and could not do. Ten more things are hidden in it that nobody wrote down,
for the children to find by accident.

The [full list is here](https://github.com/greg1232/hey-claude/blob/main/docs/capabilities.md).

It still forgets everything when it restarts, including who it is talking to.
The next things, roughly in order of how much they would improve an evening
with it:

- **Not needing the wake word every time.** Keep listening for a few seconds
  after answering, so a conversation can be a conversation. Nothing stands in
  the way of this one.
- **Interrupting it.** Half of it works — saying "I have spoken" stops it dead,
  mid-sentence. The other half is hard: with the microphone left open, the
  speaker's own voice scores 0.991 against a 0.95 line, so it interrupts itself
  about one sentence in four.
- **Hearing children better.** Small Whisper models struggle most with
  children's speech, and a misheard question is indistinguishable from a stupid
  answer. Which is a pointed problem for a speaker a six-year-old designed. The
  recordings to fix it are already in the repository.

## The code

All of it is public:

- **[github.com/greg1232/hey-claude](https://github.com/greg1232/hey-claude)** —
  the speaker itself, the two photos above, the
  [design document](https://github.com/greg1232/hey-claude/blob/main/docs/design.md)
  they turned into, everything it can do, and how the wake word is trained.
- **[huggingface.co/gdiamos/hey-claude](https://huggingface.co/gdiamos/hey-claude)** —
  the openWakeWord version of the wake word, kept for the record.
