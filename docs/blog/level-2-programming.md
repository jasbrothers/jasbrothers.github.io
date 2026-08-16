# Level 2 Programming

Tejas is six. Over a weekend he built a smart speaker: you say "Hey Claude,"
ask a question out loud, and the laptop answers out loud.

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

The finished thing is five small Python files and a loop:

```python
while True:
    wake.wait_for_wake()                     # listen for "Hey Claude"
    audio = audio_in.record_until_silence()  # record the question
    question = stt.transcribe(audio)         # Whisper, running locally
    answer = brain.ask(question)             # the Claude API
    tts.speak(answer)                        # say it out loud
```

Both requirements held. It is Python, and it runs on the laptop with no extra
hardware — the built-in microphone and the built-in speakers are the whole
device. Only one of those five steps uses the internet; the microphone, the
wake word, and the speech recognition all run on the machine, so the room is
not streamed anywhere until somebody actually says the wake word.

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

It wakes on "hey claude" at **0.997** confidence and stays under **0.005** on
ordinary conversation. It is not perfect — it likes some voices more than
others, and "hey **clyde**" scores 0.484, just under the line — but it works,
and it is a real model that did not exist before.

## So what is level 2?

Level 1 programming is telling a computer **how**. You hold the syntax, the
libraries, the file layout, the off-by-one errors. Getting good at it takes
years, and almost all of those years go into the *how*.

Level 2 is telling a computer **what** — and the striking thing is that a
six-year-old is already fluent in it. Not partially. The sketch does every job
a specification has to do:

- **It shows the product working**, from the outside, from the user's side.
- **It states the constraints that actually constrain** — Python, macOS laptop —
  and stays quiet about the hundred decisions that did not matter.
- **It marks what is unknown** instead of hiding it. *How can it connect to
  spotify?*

That last one is the most grown-up thing on the page. A question you have
written down is a question somebody can answer. The Spotify answer turned out
to be tools: Claude does not play music itself, it asks our code to, and our
code calls Spotify. It went into the design document in full — and then it got
deferred, because getting the speaker talking was worth more than getting it
singing. Knowing which one to build first is also a level-2 skill.

None of this is a claim that the *how* stopped mattering. Somebody still had to
notice that the wake word did not exist and go make one. But the ratio moved.
The scarce ingredient in that weekend was not knowing Python. It was knowing
what to ask for.

The drawing was the source code. Everything else was a build step.

## What's next

Roughly in order of fun:

- Play music by voice — the Spotify question, finally answered out loud
- Weather, timers, web search
- Remembering conversations between runs
- A light that shows when it is listening
- Moving it off the laptop and onto a Raspberry Pi, so it is a real box sitting
  on a shelf — which is what the drawing showed all along
