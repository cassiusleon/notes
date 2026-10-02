---
title: Requirements
project: "[[Home AI Assistant]]"
chapter: 1
date: 2026-10-02
tags:
  - home-ai
---

# 01 · Requirements

## What it must do
- **Lights:** adjust lights by voice or chat
- **Documents:** store important documents and send them back by email or chat on request
- **Digest:** read documents and pull out reminders and tasks (bill due dates, renewals)
- **Calendar and tasks:** create events, reminders and a task list
- **Gmail:** read past emails, including attachments
- **Lyrics:** on an Alexa request, show song lyrics on a screen, scrolling in sync with playback
- **Voice:** connect to Alexa

## Constraints
> [!important] Security perimeter
> Self-hosted model. It can't send data out, and nothing outside can query it.

- **Low running cost:** cloud hosting rejected, home hardware instead
- **Portable data:** what it learns must survive a model or hardware swap
- **Chat app:** must have Android and iOS apps. Telegram (no end-to-end encryption) and WhatsApp (too much overhead) ruled out
- **Linux** on the home box, ideally no Windows

## Design choices
- Longer tasks run in the background; instant replies aren't needed
- Alexa answers from output files that those background tasks produce

---
← [[Home AI Assistant]] | [[02 - Architecture]] →
