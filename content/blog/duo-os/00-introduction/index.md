---
title: "Why I'm Building My Own Operating System"
date: 2026-04-06
draft: false
description: "My first blog post on DuoOS, a minimal bare-metal RISC-V OS built from scratch for the Milk-V Duo."
tags: ["DuoOS", "OS", "RISC-V", "Milk-V Duo", "Bare-metal"]
categories: ["Learning"]
series: ["DuoOS"]
series_order: 0
slug: "introduction"
showTableOfContents: true
math: false
---

## The Spark

> Fortune favors the bold.
>
> — [Virgil](https://en.wikipedia.org/wiki/Fortune_favours_the_bold)

The story of DuoOS begins at [MOPCON 2025](https://mopcon.org/2025/). My junior, Brian Duan, led the scheduling team, and I was part of it. He knew [Alan Jian](https://github.com/alanjian85) from NYCU, who had previously studied at Wu-Ling Senior High School (WLSH) and was already known as an absolute beast in low-level systems. I heard they were planning to build an operating system from scratch, so I wanted Brian to introduce me. But he was too shy and thought I was being kind of weird about it, so I messaged Alan directly on Discord, introduced myself, and somehow got invited into [Libresys](https://www.libresys.net) (formerly NexOSS).

Right after MOPCON 2025, the semester was coming to an end. During winter break, Alan invited us to a meetup in Taipei called Libresys Party, and that was when he gave me this [Milk-V Duo](https://milkv.io/duo) board. The board definitely collected some dust for a while, but much later, I suddenly had a thought: what if I tried building my own operating system on this little thing?

At the time, I had originally been planning to build my own compiler, and I was working on a C parser/generator library. But from that moment on, things took a very different turn. That was the beginning of a strange and wonderful journey—and, well, the rest is history.

{{< figure
      src="milk-v-duo.png"                                                 
      alt="Milk-V Duo board"
      caption="The Milk-V Duo board that started it all"                  
>}}

## Why Reinvent the Wheel?

As of 2026, AI has become deeply embedded in everyday life. I say this not as an outsider, but as someone who actively uses it myself—I use Claude Code Max too. And yet, deep down, I know there are times when I'm not really thinking, but merely offloading cognition to AI. In that sense, it feels less like artificial intelligence and more like borrowed intelligence.

As [jserv](https://wiki.csie.ncku.edu.tw/User/jserv) once put it:

> 手機出來了，大家都不太去運動了；人工智慧出來了，大家都不太動腦子了。

Which roughly translates to: "When smartphones came along, people stopped moving as much. When AI came along, people stopped thinking as much."

Thanks to the rise of AI, the gap between a random person and someone who has spent four years grinding through computer science has, in some ways, never felt narrower. Today, anyone can open ChatGPT and type, "Build me an accounting app." The software industry was already fast-paced to begin with, but AI has accelerated it to an entirely new level.

That raises an uncomfortable question: what exactly is the difference between a random person with AI access and someone who has spent years seriously studying computer science?

I have to admit: that question hits hard. Before I started this project, my honest answer was, "Not much."

And that realization bothered me more than I wanted to admit. How could I justify being better at software engineering than a random AI user—or what people call a "vibe coder"—if I didn't truly understand the foundations myself? How could I stand out if all I could do was produce results with the help of tools that everyone else also had access to?

As Richard Feynman famously said:

> What I cannot create, I do not understand.
>
> — [Richard Feynman](https://en.wikipedia.org/wiki/Richard_Feynman)

To truly understand something, I need to be able to build it myself.

But *why* build an operating system from scratch specifically? Because of its granularity. An operating system forces you to care about everything. It pushes you closer to the machine than most software ever will. A single mistake can crash the system, reboot the machine, or corrupt data. There is very little room to fake understanding.

And that is exactly why I chose it. Only by trying to build an OS have I started to appreciate just how much modern operating systems abstract away—the messy, fragile, hardware-facing details that most of us rarely have to confront directly. Building one means stripping away that comfort and facing the raw machine more honestly.

That, ultimately, is why I started this project. I don't want to depend on borrowed intelligence forever. I want to develop the kind of understanding that makes me genuinely different—the kind that is harder to imitate, harder to replace, and earned through real thought.

Hopefully, through this kind of training, I can become someone set apart by real understanding.

## What DuoOS Actually Is

DuoOS is a minimal RISC-V operating system for the [Milk-V Duo](https://milkv.io/duo), built primarily as a learning project.

I want to build it the hard way, with as little abstraction as possible—but not recklessly hard. For now, I am accepting a few "training wheels," namely OpenSBI and U-Boot, so that my first pass at OS development does not collapse under too much upfront complexity. I want the challenge to be real, but still bearable.

At its core, DuoOS is a C operating system for RISC-V, built to answer a simple question: how does an operating system actually work under the hood? It may remain minimal. It may never become complete. But that is not really the point. The point is to learn by building.

And that brings me to how this project actually unfolded—through three separate attempts.

## How It Actually Went

> Action without thought is empty. Thought without action is blind.
> 
> — [Kwame Nkrumah](https://www.goodreads.com/quotes/901425-action-without-thought-is-empty-thought-without-action-is-blind)

I am a world-class procrastinator. The dangerous thing about procrastination is that it is really good at disguising itself as productivity. You tell yourself you're just looking for the best resources, the best roadmap, or the best way to begin. Before you know it, you're deep in analysis paralysis, genuinely believing you're putting in the work, when in reality it's just low-effort busywork. But this time, I told myself, it was time to change that.

### The First Attempt

My first attempt began on March 25, 2026. It was a sunny afternoon. I looked at my Milk-V Duo sitting on the shelf and thought, "Maybe I can do something more than just let it sit there collecting dust."

Then one idea hit me: "Let's build an OS on it."

But how? Like any ordinary person living in 2026, I turned to AI for help—namely Claude. After a fair amount of tweaking and fighting with it, I managed to build the very first version of DuoOS. But right from the beginning, I knew deep down that I did not truly understand what the hell I had built. That realization led to the second attempt.

{{< figure
      src="first-attempt.png"                                                 
      alt="First Attempt on DuoOS"
      caption="First Attempt on DuoOS"                  
>}}

### The Second Attempt

AI has both pros and cons. Used wisely, it can greatly improve both learning and productivity. This time, I tried to use it more deliberately.

The next day, March 26, 2026, I used Claude for deeper research, and that eventually led to a major revamp of DuoOS. It generated a long research guide on how to build an operating system from scratch. Through that process, I started to understand much more of the actual path from bare metal to a working system: linker scripts, ELF loading, device trees, image layout, how U-Boot loads a program, and many other foundational details. I learned a lot from that stage.

After the initial setup, I could at least say that I had managed to run a bare-metal program on the Milk-V Duo. That alone already felt like a small win. But that was still far from an actual operating system. So I moved on to writing a basic shell and a `printf`-like function—yes, you read that correctly, because `printf` does not simply exist for you in a bare-metal environment.

After that, I started implementing more features: boot-time information, LED control, timer support, hart-related commands, watchdog timer handling, trap handling, timer interrupts, and an uptime command.

But during debugging, another thought hit me: why not document all of this? At the same time, I also realized the codebase already contained too many bugs, too many bad decisions, and too much messy code. So I decided to start over again.

As the saying goes, third time's the charm.

{{< figure
      src="second-attempt.png"                                                 
      alt="Second Attempt on DuoOS"
      caption="Second Attempt on DuoOS"                  
>}}

### The Third Attempt

I have not started the third attempt yet, but I already know this time will be different. I asked Claude to generate a more thorough and better-structured guide for building the system step by step, all the way toward networking.

The scope is honestly a little wild: UART I/O, trap handling, virtual memory, process scheduling, user mode, `fork`/`exec`, a FAT32 filesystem, an ELF loader, Unix pipes, and eventually a working shell—all the way from bare metal upward.

That is the version I plan to document in the next series of blog posts.

> Note: The reason this is only being written on April 6 is partly my own world-class procrastination, and partly the usual interruptions from university quizzes and coursework.

## Conclusion

> Be the designer of your world and not merely the consumer of it.
> 
> — [James Clear](https://jamesclear.com/quote/atomic-habits)

From here on, I'll be documenting the entire journey in this series. If you want to follow along, check out the [DuoOS series](/blog/duo-os).

## Disclosure

AI was used to help refine the writing. All ideas, experiences, and original content are my own.