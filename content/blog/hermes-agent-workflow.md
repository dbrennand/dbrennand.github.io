---
title: "My Hermes Agent Workflow: one agent, every device"
date: "2026-09-04T08:23:50+01:00"
tags: ["Hermes Agent", "AI"]
draft: false
---

About two months ago, I tore down my previous [Hermes Agent](https://hermes-agent.nousresearch.com/docs) deployment and redeployed it from scratch. Since my last blog post [Argus: My personal AI assistant powered by Hermes Agent](../hermes-agent), I've learned a lot about Hermes and changed how I use it. In this blog post, I want to share my workflow.

My Hermes Agent runs on a [Hetzner](https://www.hetzner.com/cloud/) VPS, but I interact with the same agent from Discord, Telegram, my MacBook, and my Windows PC. Tailscale connects my MacBook and Windows PC, so I don't need to expose the [Hermes remote backend](https://hermes-agent.nousresearch.com/docs/user-guide/features/web-dashboard#connecting-hermes-desktop-to-a-remote-backend) to the internet.

## Why Hermes Agent?

So, why Hermes Agent, I hear you ask? :wink: Here are some of the reasons.

### Open source and fast development

The [project](https://github.com/NousResearch/hermes-agent) is evolving rapidly, with new features added constantly, sometimes multiple times a week. There is always something new to try, which I like, and it is also open source under the [MIT license](https://github.com/NousResearch/hermes-agent?tab=MIT-1-ov-file).

### Freedom to choose model providers

Hermes Agent doesn't tie you to one model provider or API. You can choose providers based on capability, cost, speed, availability, and the type of task you're doing. This was the biggest selling point for me because I can change the underlying model that Hermes uses while keeping the interface I interact with the same.

### Memory that adapts to your workflow

One key element of any useful agent is its memory. It remembers your project context and preferences, avoids asking repeated questions, and becomes more useful as you work together. Hermes Agent does this very well. However, it is by no means perfect. You still need to review and correct memories over time, but you can do that by having a conversation with the agent. :sunglasses:

### Control over where data is stored

The [freedom to choose model providers](#freedom-to-choose-model-providers) means you can use cloud-based and local models. Hermes Agent can integrate directly with models running via [Ollama](https://hermes-agent.nousresearch.com/docs/guides/local-ollama-setup) and supports cloud and local [memory providers](https://hermes-agent.nousresearch.com/docs/user-guide/features/memory-providers/#available-providers) too, giving you full control over where your data goes.

## My setup at a glance

My Hermes Agent is deployed on a Hetzner CX23 VPS. The desktop applications connect to the Remote Gateway through a [Tailscale](https://tailscale.com/) VPN tunnel, while Discord and Telegram provide chat interfaces that I use whilst on the go. [Mnemosyne](https://github.com/mnemosyne-oss/mnemosyne) stores the agent's memories locally in an SQLite database.

```goat
+--------------------+      +--------------------+
| MacBook + Windows  |      | Discord + Telegram |
| Hermes Desktop Apps|      | Messaging clients  |
+---------+----------+      +---------+----------+
          |                           |
+---------v----------+      +---------v----------+
| Tailscale VPN      |      | Discord / Telegram |
| tunnel             |      | platform APIs     |
+---------+----------+      +---------+----------+
          |                           |
          +-------------+-------------+
                        |
              +---------v---------+
              | Hetzner VPS       |
              | Hermes Agent      |
              | Remote Gateway    |
              | Messaging Gateway |
              +---------+---------+
                        |
          +-------------+-------------+
          |                           |
  +-------v------+              +-----v--------+
  | Model        |              | Mnemosyne    |
  | providers    |              | SQLite       |
  | ChatGPT/Nous |              | local memory |
  +--------------+              +--------------+
```

## Deploying Hermes on Hetzner

My Hermes Agent is deployed on Hetzner using [hermes-on-hetzner](https://github.com/dbrennand/hermes-on-hetzner/tree/main). The project uses Ansible to deploy the VPS and install Hermes Agent. Tailscale integration is [optional](https://github.com/dbrennand/hermes-on-hetzner/tree/main#optional-tailscale-connectivity), but I highly recommend it. See the [repository README](https://github.com/dbrennand/hermes-on-hetzner/tree/main#%EF%B8%8F-configuration) for all the configurable variables.

Below is the high-level deployment workflow once you've cloned the repository and configured the variables under `vars`:

```bash
uv sync
uv run ansible-galaxy collection install \
  -r collections/requirements.yml \
  -p ./collections

export TAILSCALE_AUTHKEY="***"
export HCLOUD_TOKEN="***"
uv run ansible-playbook playbook.yml
```

## Model providers and subscriptions

I use ChatGPT Plus via OAuth to access OpenAI models and Nous Research to access models from multiple providers. At the time of writing, the Nous Research service has 383 models to choose from :exploding_head:. I pay £20 per month for ChatGPT Plus and $20 per month for my Nous Research subscription.

I use a mixture of models. For quick questions and research, I use smaller models via Nous Research, such as DeepSeek V4 Flash 0731 or GLM 5.3 Flash. For complex coding, I generally use OpenAI's GPT 5.6 Sol (medium effort) as the planner and GPT 5.6 Luna (high effort) for the coding subagents.

> [Referral Link](https://portal.nousresearch.com/r/dbrennand): Nous Research often runs promotions and provides a selection of free models. If you decide to try it, my referral code gives you $15 off your first month, and I receive $10 in credits.

## How I interact with Hermes

I use Discord and Telegram for quick requests. Discord is my main messaging channel. Telegram is a backup if Discord is unavailable, and Discord is the fallback if Telegram is unavailable.

For longer sessions, I use the Hermes Desktop apps on my MacBook and Windows PC. They connect to the Remote Gateway, so I can switch devices without losing project context or preferences.

| Interface           | How I use it                                                        | Best for                                   |
| ------------------- | ------------------------------------------------------------------- | ------------------------------------------ |
| Discord             | Mention `@Hermes Agent` when I have a quick request                 | Quick requests                             |
| Telegram            | Use a dedicated chat for quick requests and as a backup for Discord | Quick requests when Discord is unavailable |
| macOS desktop app   | Connect to the Remote Gateway through Tailscale                     | Longer, focused sessions on my MacBook     |
| Windows desktop app | Connect to the same Remote Gateway through Tailscale                | Longer, focused sessions on my Windows PC  |

## Mnemosyne as the memory provider

I chose [Mnemosyne](https://github.com/mnemosyne-oss/mnemosyne) as the memory provider because it was simple to install, is self-hosted, requires no external service, and stores the memory data on the VPS. Other providers (such as the [hosted Honcho service](https://honcho.dev/)) store the agent's memories on their service. Although Mnemosyne stores memories locally, that doesn't mean every conversation stays on the VPS. When Hermes uses an external model provider, the request, along with the relevant memories, is still sent to that provider.

Mnemosyne remembers my preferred tools, communication style, and workflow decisions, such as cloning repositories with [gitclone](https://github.com/dbrennand/dotfiles/blob/main/dot_local/bin/executable_gitclone), because I like to keep things organized.

## Typical session

A typical session for me looks like this:

1. Before starting a session, I choose the model I want to use based on the task.
2. On my MacBook or Windows PC, I turn on Tailscale and launch the Hermes Desktop app. Hermes is already running and ready to go on the VPS.
3. I give Hermes the project context and ask it to inspect the repository or conduct some research before we begin.
4. We create a project plan and work through the implementation.
5. When I leave the desktop app, I can send a quick follow-up through Discord or Telegram.
6. When I return to the desktop app, I continue using the same session with the task and project context intact.
7. If I make a workflow decision that will be useful later, I ask Hermes to remember it. More often than not, Hermes does this itself, which is really nice. :sunglasses:

## Trade-offs and rough edges

The main trade-off is maintaining the VPS. I need to update Hermes manually and keep the deployment healthy. The initial setup takes some time, especially configuring model providers and credentials, but those are one-time tasks. After that, the ongoing work is mainly applying updates and dealing with the occasional configuration issue.

The VPS is also a single point of failure. If it goes down, so does Hermes. That is an acceptable trade-off for me.

## Conclusion

Hermes gives me several ways to communicate with the same agent, as well as the flexibility to choose the model I want for each task. I can use the desktop app for a longer session, send a quick request through Discord or Telegram, and return to the desktop later without starting over.

The way Hermes preserves memory and context is what makes it especially useful. Hermes can carry relevant project context across sessions and remember how I prefer to work, including tooling choices, communication style, and workflow decisions. It still needs correcting from time to time, but it becomes more useful as it learns my preferences.

If you are on the fence about trying Hermes, I highly recommend giving it a try. Being able to choose the interface and model that suit the task, while keeping my project context and preferences available, has made Hermes a valuable part of my workflow.
