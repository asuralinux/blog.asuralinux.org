---
title: Hello World
date: "2023-10-01"
author: 'Gaveen Prabhasara'
categories:
  - Announcement
  - Update
  - Project Meta
tags:
  - meta
  - community
  - faq
  - foss
---

Hello there! Welcome to the Asura Linux Project. This is the first of hopefully many posts in the fresh new Asura Linux blog. It almost smells like fresh paint and a just-mowed lawn here.

I recently started Asura Linux as a new Open Source project to explore the possibilities of building modern Linux systems with emerging foundational userspace technologies and concepts—ultimately guided by user experience. In simpler words, I want to experiment with different ways of building and running Linux systems. My reasoning is simple—I want to build the Linux user experience I want to use.

I am Gaveen, a sysadmin turned engineering manager, a programming language tourist, and a petty dabbler of the written word. That is what my public profiles generally say. I've been using Linux personally and professionally for more than two decades. However, I have never been involved with building a Linux distribution—at least not anything that wasn't a derived customization of an established distro. I have never written non-trivial code to make a Linux distribution (distro) operationally viable. Not a lot of resources talk about this type of infrastructural work either. How should the deterministic build environments be set up? Which kernel build parameters go into a release kernel? Which stages of the boot process can be measured? What would even an image-based delta update system look like? How can the different expectations of application availability channels be reconciled? How well would BTRFS snapshots play with encrypted home directories? How practical is it to use a finer outer layer granularity with OverlayFS? What other undiagnosed complications might I find? I don't have all the answers to seemingly fundamental questions in this space. Therefore, I'll be the first to admit I'm out of my depth here.

Luckily, not having the answers hasn't stopped me before. I am driven by the desire to use the kind of Linux user experience I wish for. Naturally, I am excited to work on it—learning along the way and sharing what I've learned. This is why Asura Linux exists.

For example, the next blog post (coming in the next couple of days) will explain the Linux boot process—the past, present, and future. We'll start with how a Linux system historically booted (e.g., in a BIOS-based system with MBR disk partitioning and SysV init), then how it works these days (e.g., in a UEFI-based system with a GPT disk partitioning and systemd), and finally what could be possible in the immediate future (e.g., fully-fledged Secure Boot with systemd and friends). But I digress.

While the "what I want to use" is always a moving target, there have been enough exciting recent developments in the Linux world that shed preconceived notions about how Linux distributions are put together. Coincidentally, nuanced approaches such as immutable, image-based, secure/measured-boot, or managed distros are working towards goals with enough common ground to warrant collaboration in building robust Linux systems.

Let's take a sample scenario. An immutable/image-based Linux system tends to have a system partition that does not change on disk during its operation (i.e., immutability). It updates in a single task that—in its entirety—either succeeds or fails without resulting in a partial/unknown update status (i.e., atomic). It also means there is always a known previous working state to revert to (i.e., rollbacks) if anything goes wrong. If you have been paying attention to Linux distribution development, you may have seen several popular Linux distros already offering an immutable flavor or considering one. These include Fedora Silverblue, openSUSE Aeon, Ubuntu Core Desktop, GNOME OS, and Flatcar Container Linux.

Stakeholders from these projects and more have started to build consensus on specifications. For example, the first Image-based Linux Summit took place almost a year ago in Berlin. Its participants formed the Linux Userspace API (UAPI) Group[^uapi] and formalized a few specifications to collaborate around. They just had a follow-up in September 2023 with even more participants. It's definitely an exciting time to work in this space.

On the other hand, even with many distros participating and a collaboration effort underway, image-based/immutable Linux is not a solved problem. The current workflows and the desired expectations have a vast gap between them. To make the Linux user experience accessible to more people, we need to address that UX gap and other technological edge cases. There is a lot to be done, and sometimes, entirely new kinds of software experiences to be developed. This is where the Asura Linux Project comes in—for me.

Hopefully, it'll also be a project of interest for you soon. While we are not there yet, I hope for a kind community around Asura Linux when we can actively collaborate.

So, where do we begin? Let's start with answering a few questions you might have.

## Who are "we" in the project?

Since this is still a single-contributor project I work on in my free time, I am the only active contributor. There is a set of friends and colleagues I consistently subject to info-dumping about the project who have been kind enough to share their opinions and expertise with me.

As mentioned, I aspire to have a healthy Open Source community around Asura Linux. In that spirit, I will refer to the open source project as we/us going forward.

## Is Asura Linux a distribution?

Not necessarily. While Asura Linux will build a distribution (distro) as a way to experiment, it's safe to say you won't see a distro for end users for a good while. Therefore, next year's builds will primarily be for people who wish to develop and contribute to Asura Linux. We'll be generating OS images, but these are meant to be experimental during this period.

## Is Asura Linux yet another distro?

No. Asura Linux is not trying to build a derived customization of an existing Linux distro. It will take an experimental approach to research and develop around a set of technologies and concepts.

## Is Asura Linux based on another distro?

No, and Yes. Testing our resource and capacity limits by trying to build an entirely original Linux distro from scratch by pulling software directly from upstream projects is not pragmatic since we know that will fail. We will use other Open Source projects and have an upstream-first contribution policy. Therefore, as a start, our experimental distro will use software repositories from the Fedora Project to the extent it makes sense for the project goals.

## Will Asura Linux be based on Fedora?

Yes and No. As explained above, Asura Linux will base our first distro experiment on the software repositories from the Fedora Project. Should we meet the requirements defined by the Fedora Project, it may be called Fedora Asura Remix.

By remaining as a Remix, we could still deviate from Fedora components and conventions based on our requirements. For example, we don't intend to use a package manager like DNF (like Fedora Workstation or Server) as a user-facing component, nor are we planning to use OSTree (like Fedora Silverblue or CoreOS). At the same time, we intend to keep SELinux in a targeted enforcing configuration.

## When can I use the distro?

We plan to have automated builds in place within a few weeks. These will generate images that can be booted as a VM. This infrastructure will be necessary as it allows us to iterate experiments faster. However, it'll take a few months for an opinionated Asura Linux to take form.

There is no current timeline considered to have releases targetting end-user consumption. It'll be a future decision.

## Will the Asura Linux experimental distro be immutable?

Yes—generally speaking. As of this writing, we think it'll be an image-based Linux, for the lack of a better term. This means the base system and subsequent updates will use an image-based mechanism instead of individual package updates.

## What other features are planned?

There are some aspirational characteristics planned for the Fedora Asura Remix. As of this writing, the more prominent of them are as follows.
- Image-based system and updates
- Compliance with relevant UAPI Group specifications[^uapi-specs] (e.g., DPS/DDI, UKI, SysExt, ConfExt, etc.)
- System integrity verification (e.g., dm-verity)
- Diverse application channels (e.g., Flatpaks, OCI Containers, etc.)
- Secure boot (if achievable under meaningful conditions)

However, it should be noted that any of these can change over time.

## Would there be a version without systemd?

No. Despite some vocal criticism from a minority of people, systemd remains one of the fundamental infrastructure software in the modern Linux landscape. Therefore, there is no plan whatsoever to avoid systemd. While we don't have plans to implement all the different components of systemd, we don't want the use of systemd to be a point of contention either.

## What promises do you make as an Open Source project?

- Asura Linux will be open source and always remain so.
- Asura Linux will be a friendly, inclusive, and welcoming community with a Code of Conduct.
- Asura Linux will collaborate with other open source projects rather than compete with them.
- Asura Linux will value Integrity, Decency, Responsibility, Inclusion, and Excellence.

## How can I support/contribute?

As mentioned earlier, we are not ready to accept contributions just yet. Sometime soon, but not just yet as a single-developer project. I want to find my bearing around the intended features and set up the necessary infrastructure first.

Therefore, we also don't have many repositories (repos) in the open yet. I intend to publish the build repos and software components to be developed in the upcoming weeks. From that point onward, all project activities will happen in the open, accepting community-driven contributions.

Until then, please keep an eye out for this blog. I plan to write a series of posts explaining the things we learn along the way and explaining the technical decision-making. These posts will start from our next post on the Linux boot process.

In the meantime, you can also check the—somewhat outdated—[meta documentation](https://github.com/asuralinux/asuralinux) repository on GitHub. The scope section particularly needs updating.

I am going through the information from the latest Image-based Linux Summit right now. In addition, there's an excellent list of talks from All Systems Go! 2023 conference[^asg23]. As I catch up on these new developments, I will gradually update the meta documentation—right after the next blog post.


[^uapi]: [The Linux Userspace API (UAPI) Group](https://github.com/uapi-group/)
[^uapi-specs]: [UAPI Group specifications](https://github.com/uapi-group/specifications)
[^asg23]: [All Systems Go! 2023 talks YouTube playlist](https://www.youtube.com/playlist?list=PLWYdJViL9EioDNHn7xIqQJLyCayNPKeYf)
