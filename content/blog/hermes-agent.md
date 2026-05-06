---
title: "Argus: My personal AI assistant powered by Hermes Agent"
date: 2026-05-06T07:57:15+01:00
draft: false
---

Like many folks these days I'm using AI at work and outside of work to help me with various tasks. Some recent uses include: coding, creating policy and process documents for the charity I volunteer at, troubleshooting technical issues and many more. The applications I've used so far being [Claude Code](https://claude.com/product/claude-code), [Codex](https://openai.com/codex/), [Claude Desktop](https://claude.com/download) and [Cursor](https://cursor.com/).

Using all these tools had recently got me thinking:

- How could I use AI to help improve my day to day habits?
- Could I use AI to provide me some actual quality of life improvements?

Late last year (~November 2025) [OpenClaw](https://openclaw.ai/) took the tech world by storm. If you're not aware [OpenClaw](https://openclaw.ai/) is a free, open source (MIT Licensed) AI personal assistant which uses chat platforms like Telegram, WhatsApp and others to communicate with it. You can integrate it with various services, for example your Google account so it can read your emails, and respond to them on your behalf.

At that time it seemed like all my social media feeds were flooded with information about it, the productivity benefits it was providing and also the security concerns. The latter mainly putting me off.

Since then in recent weeks I heard about a similar project called [Hermes Agent](https://hermes-agent.nousresearch.com/docs) from [Nous Research](https://nousresearch.com/). It's also free, and open source under the MIT license but what caught my eye was that it has a ["built-in learning loop"](https://hermes-agent.nousresearch.com/docs#key-features) meaning the agent learns your habits and preferences from its experiences interacting with you, nudging itself to persist knowledge and even create [skills](https://agentskills.io/home) to become more efficient the next time you ask it to perform a task.

## Argus - My personal AI assistant powered by Hermes Agent

I deployed [Hermes Agent](https://hermes-agent.nousresearch.com/docs) onto a Hetzner Cloud VPS using Ansible with my project [`hermes-on-hetzner`](https://github.com/dbrennand/hermes-on-hetzner). Just like that; Argus was born!

After some back and forth with GPT-5.4 I landed on this name because:

> Argus Panoptes ("Argus the all seeing") was a giant with countless eyes all over his body, whom the goddess Hera employed as a guardian.

So Argus is my all seeing guardian helping me with my daily tasks. Also it's kind of ironic because:

> Hermes, the messenger god, was sent by Zeus to rescue Io from her imprisoned state. Hermes killed Argus by lulling him to sleep.

A small little easter egg too :grin:

The cloud model I'm using to power Argus is `gpt-5.4-mini` as I already have a ChatGPT Pro subscription and the model has been more than capable so far!

I've been using Argus to manage my schedule on Google Calendar, run a weekly review which is basically where we plan the week ahead, meal ideas for each day taking into account my schedule, and based on that create a shopping list of ingredients ready to go for grocery shopping. I've also used it to set reminders, provide me a daily news digest every morning and research hotel options for going away.

I was initially put off of these AI assistants because of the security concerns, so right now I've only granted Argus the ability to read and create events on my calendar. I don't think I'm *quite* ready to give it access to my personal files just yet.

## Future Work

In the future I plan to explore how I can implement some guardrails around the agent's access to certain files. For example, if I did give it access to my Google Drive, I'd only want it to have access to certain folders. Furthermore, I've seen others using their agent to automatically triage GitHub issues and review pull requests, so I'd like to see if I can do something similar with Argus. Finally, I'm starting to build up a library of skills which are probably useful to others, so I'm going to look at publishing those to a GitHub repository too.

---

If you're doing cool stuff with Hermes Agent or OpenClaw then let me know via [X](https://x.com/dbrenuk) or [Email](mailto:contact@danielbrennand.com). Until next time! :wave:
