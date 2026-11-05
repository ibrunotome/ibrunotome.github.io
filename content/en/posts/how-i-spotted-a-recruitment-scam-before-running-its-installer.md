---
title: "How I Spotted a Recruitment Scam Before Running Its Installer"
url: how-i-spotted-a-recruitment-scam-before-running-its-installer
date: 2026-11-05T11:29:09-03:00
tags: ["security", "macos", "scams", "career"]
categories: ["security"]
draft: false
translationKey: "how-i-spotted-a-recruitment-scam-before-running-its-installer"
---

Fifteen minutes before a scheduled job interview, I received a link to an application I was supposed to use for the meeting.

The recruitment process was associated with a company called Saifuro. The interview application was called Vettasy. Both had websites, documentation and enough professional-looking content to make the process seem plausible.

I did not run the installer. I looked at what it wanted to do first.

That decision led to an investigation of a shell script that downloaded and immediately launched a Mac executable. AI-assisted static analysis found strong matches to a documented malware family called CloudSyncD.

Here is what happened, what made me suspicious, and what the evidence actually established.

## Why I was already skeptical about Calendly links

A few weeks earlier, I had read [Fabio Akita's article about an attempted scam involving Calendly](https://akitaonrails.com/2026/09/22/pseudo-techcrunch-tentou-me-pegar-em-golpe-calendly-e-uma-merda/).

In that case, someone posing as a TechCrunch journalist approached him for an interview. He described a genuine Calendly link that eventually led to an X authorization request for a malicious third-party application.

Reading that made me more skeptical when a Calendly link appeared in an unfamiliar conversation. A meeting invitation can look normal while being one step in a harmful process.

My case involved a different mechanism: an interview-software download. I did not establish that the two incidents had the same operators. What I remembered from his article was to look beyond the scheduling tool and check what came next.

## How the recruitment process reached me

The role was for a Founding Full-stack Engineer, and the package stood out: **a US salary of $190,000–$225,000 per year, 1% equity and several benefits**. The stack also matched what I have been working with since June 2025. It was an attractive combination of compensation and a fit with my experience.

The sequence was:

1. Contact through Indeed or `contact@saifuro.com`.
2. A referral to the Telegram account `@Ithelia_Dias`.
3. A Calendly link to schedule the meeting.
4. A Vettasy link arriving approximately 15 minutes before the meeting.

During the investigation, Saifuro's website was found to publish the same contact email and to link the Telegram account to the person it presented as its founder and CEO, Ithelia Dias.

This connected the conversation to contact details advertised by the website. It did not establish the legal identity of the person using the account, whether an account had been compromised, or who operated the malware infrastructure.

The Telegram account was later deleted. To capture how the profile had looked before, I opened the app without a network connection and took the screenshot below, which still showed the locally cached name and profile photo.

<figure>
  <a href="/saifuro-telegram-perfil-ithelia-dias.webp">
    <img src="/saifuro-telegram-perfil-ithelia-dias.webp" width="900" height="180" loading="lazy" decoding="async" alt="Telegram profile showing the name Ithelia Dias and the photo still visible when opening the app offline.">
  </a>
  <figcaption>Profile cached in the offline app, before it updated to show the account as deleted.</figcaption>
</figure>

In the reply about scheduling the interview, the person described an introductory call of up to an hour, asked for a short introduction and sent the Calendly link. The message already stated that the meeting link would arrive only 15 minutes before the scheduled time.

<figure>
  <a href="/saifuro-telegram-agendamento-entrevista.webp">
    <img src="/saifuro-telegram-agendamento-entrevista.webp" width="758" height="1784" loading="lazy" decoding="async" alt="Telegram conversation with interview instructions and a Calendly link to schedule the November 3 meeting.">
  </a>
  <figcaption>Reply with instructions and scheduling details. In this screenshot, Telegram already showed the account as deleted.</figcaption>
</figure>

On the day of the interview, November 3, I questioned Vettasy, mentioned that I had found at least three red flags in the installation script and suggested Google Meet or another browser-based platform. The reply came within seconds, with a long message and a ready-made explanation. Given how quickly it arrived, my impression was that the message was already on the clipboard, waiting to be pasted.

The reply justified the app as a way to prevent AI-generated answers and said that sending the link only 15 minutes in advance would stop candidates from finding ways around that measure. After I pressed the concerns about the script, the person replied that they were discussing switching platforms.

<figure>
  <a href="/saifuro-telegram-dia-entrevista-vettasy.webp">
    <img src="/saifuro-telegram-dia-entrevista-vettasy.webp" width="754" height="1128" loading="lazy" decoding="async" alt="November 3 conversation in which I question Vettasy and receive an explanation for sending the link 15 minutes before.">
  </a>
  <figcaption>Conversation on the day of the interview, after I pointed out problems with the installer and asked for another platform.</figcaption>
</figure>

The timing of the download mattered to me. When a meeting is about to start, there is pressure to get the application working and avoid being late. That is a bad moment to make a decision about running unfamiliar software.

## Behind the professional appearance

Saifuro presented itself as infrastructure for AI agent payments. Vettasy presented itself as an interview platform with recording and session-verification features.

You can view [a November 3, 2026 snapshot of Saifuro's website on the Internet Archive](https://web.archive.org/web/20261103193517/https://saifuro.com/) and [Saifuro's documentation archived on the same date](https://web.archive.org/web/20261103194115mp_/https://docs.saifuro.com/). They even had [a Medium profile](https://medium.com/@Saifuro), with posts dated as far back as August: almost three months of history by the time this process happened.

The [careers page, also archived on November 3, 2026](https://web.archive.org/web/20261103193535mp_/https://saifuro.com/careers), advertised **22 open positions**. That reinforced the appearance of a growing company, with a hiring process extending beyond the role offered to me.

Both the landing page and the Medium articles mentioned x402, an HTTP payment protocol that had recently gained the backing of the Linux Foundation. The [x402 Foundation officially began operating under the Linux Foundation in July 2026](https://www.linuxfoundation.org/press/linux-foundation-announces-operational-launch-of-x402-foundation-to-standardize-internet-native-payments-for-ai-agents-and-applications). Referring to a real, recent technology associated with a well-known institution helped give Saifuro's pitch credibility.

What stood out to me was how sophisticated the presentation was. Websites, documentation, articles, profiles and a recruitment conversation with clearly defined steps all reinforced each other's credibility. Putting that together takes time and care; the posting history also gave the impression of a company that had been active before the approach. The attractive offer came with an entire public presence to support it.

The investigation turned up product explanations, privacy policies, security claims and named profiles. There were also real public records associated with the Saifuro identity: a [W3C community-group participation record](https://www.w3.org/groups/cg/agentprotocol/activity/) and an [individual Internet-Draft submitted to the IETF](https://datatracker.ietf.org/doc/draft-dias-agent-payment-verdicts/).

Those records have limited meanings. A participation record shows participation under a name and affiliation. An individual Internet-Draft is a submitted document; the IETF page explicitly states that it is not an endorsement or an approved standard.

I also investigated Vettasy manually. In my Google searches, I found only **two pages about it** and could not find a single backlink, meaning a link from another website pointing to the platform. The public footprint I could find was minimal for an app I was expected to install and use for an interview.

The investigation with Codex also uncovered a contradiction in the security claims: one page presented Vettasy as already having SOC 2, while another said it would only arrive in **Q4 2027**. Those two versions were incompatible and made me more suspicious of the assurances published on the site.

None of that answered the question I needed answered: what would this installer run on my computer?

## How the installer was inspected

I used Codex to help investigate the websites and inspect the installer. My instruction was explicit: **do not execute it**.

The shell script was retrieved as text. The archive it referenced and the executables inside it were inspected as data using Python and tools such as `file`, `nm`, `otool` and `codesign --display`. Neither the shell installer nor the downloaded executables were launched. The recovered command-and-control endpoint was not contacted.

This was static analysis. It could identify code, configuration and known malware fingerprints, but it could not demonstrate every runtime behavior or determine what additional payload a remote server might provide.

The findings below come from the sample retrieved on **November 3, 2026**. They are an AI-assisted investigation, not a claim that I independently performed a complete malware reverse-engineering exercise.

## What was wrong with the shell script?

The script had several concrete problems.

### It disabled certificate verification

The payload-download command included:

```bash
curl -fsSLk -o "${_sys_dest_6vm}" "${_app_url_dyau}" </dev/null
```

The `-k` option disables TLS certificate verification. That weakens the authentication of the connection delivering the executable.

That flag alone would not establish malware. It was one part of the evidence.

### The signature-verification function was never called

The script defined a GPG signature-verification function, but the active installation path never called it. There was no expected checksum or verified publisher identity checked before execution.

A verification function sitting elsewhere in a script does not verify the file being installed.

### The progress bar was driven by a timer

The displayed installation progress advanced through a loop with short sleeps. It did not measure download or extraction progress.

A decorative progress bar is possible in legitimate software too. Here it added to the concerns about what the installer was presenting to the user.

### It launched a discovered executable immediately

After downloading and extracting the archive, the script searched for an executable and passed it to:

```bash
exec "${__bin_bin_sy}"
```

There was no pause to inspect the downloaded program or confirm its publisher. Running the installer would hand execution directly to that program.

The archive retrieved during the investigation contained a single file named `installer`: a universal macOS executable supporting both Intel and Apple Silicon.

## What connected the executable to CloudSyncD?

The binary analysis produced stronger evidence than the website's appearance or the shell script's poor practices.

| Finding | Observed evidence |
| --- | --- |
| Malware-family fingerprint | The same 32-byte string-obfuscation table documented for CloudSyncD. |
| Matching configuration | The C2 encryption key and IV published in CloudSyncD research. |
| Credential-related code | Password-dialog and password-saving functions, including `cloudsync_impl::save_local_password`. |
| Local password validation | Decoded strings naming `dscl`, `/Local/Default` and `-authonly`. |
| Additional executable | A separate universal Mac executable embedded inside the downloaded installer. |
| Publisher identity | An ad hoc code signature with no Apple developer Team ID. |

These findings closely matched the family described by [Jamf Threat Labs](https://www.jamf.com/blog/cloudsyncd-macos-backdoor-fake-zoom-installer/) and [Elastic Security Labs](https://www.elastic.co/security-labs/threat-command/apex-accelerator-malicious-npm-packages). Their research describes password-phishing components and a backdoor capable of receiving further executables.

The assessment was a **high-confidence CloudSyncD-family match**. It was based on shared code and configuration fingerprints, not an assertion that the entire file had the same hash as a sample in those reports.

Neither research report independently attributed this recruitment process to Saifuro or the person behind the Telegram account. That connection came from my recruitment experience and the particular Vettasy download inspected here.

For anyone correlating reports, the downloaded `installer` executable had this SHA-256:

```text
782f21b0ce945931bab8b1d84dc57d542a510ec66b80f9034fefd04dcd53572b
```

The configured endpoint recovered from the embedded payload was:

```text
hxxps://dodiihome[.]com/macos/jquery.js
```

That address is deliberately defanged. It is an indicator for investigation, not a link to visit.

## What surfaced in the following days

On **November 3**, I had already refused to run the installer. The records below appeared after that decision and help document what became public in the following days.

On **November 4**, a [review titled “Probably an elaborate scam”](https://app.ethos.network/activity/review/722566/probably-an-elaborate-scam) was published on Ethos Network, raising concerns about Saifuro. Also worth documenting is the [profile associated with the username `Ed_Wynand`](https://app.ethos.network/profile/x/Ed_Wynand), who presented himself as the company's CTO.

While following the case through **November 5**, I also found [CryptoJobsList's public announcement](https://t.me/cryptojobslist/11669) stating that it had reviewed reports about Saifuro, removed the associated job listings and banned the account.

I reported the job to Indeed as a fake listing associated with malware delivery. It has since been removed.

## What I learned

Nobody needs to rush into installing just anything because a meeting starts in X minutes. Reading about someone else's experience made me pause long enough to check, and that pause helped.
