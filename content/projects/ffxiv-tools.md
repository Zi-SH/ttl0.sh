---
title: "ffxiv-tools"
archived: true
weight: 202
---
**STATUS:** `Abandoned`

A BASH-based tool for configuring Final Fantasy XIV and ACT on Linux.

Source: https://github.com/Zi-SH/ffxiv-tools
<!--more-->
---

Prior to the creation of [xlcore]( {{< relref "projects/xlcore" >}}), manual configuration of Final Fantasy XIV was required on Linux. While its configuration wasn't notoriously difficult allowing other apps to work along-side it on the same WINESERVER was often complicated. 

When I began doing hardcore raiding on Final Fantasy XIV, I required ACT (Advanced Combat Tracker) for performance metrics I could review after the content. This application required fairly specific networking configurations, especially under Linux, due to things such as networking capabilities on binaries. More specifically, features like `cap_net_raw`, `cap_net_admin`, and `cap_sys_ptrace=eip` were required on the WINE binaries for the functionality ACT provided.

This clusterfuck has been superceded by both GoatCorp's XIVQuickLauncher (XLCore) and Marzent's IINACT (It Is Not ACT) Dalamud Plugin.