# Dara Setup Guide

Dara is a deal desk. In a small business the owner is the salesperson, the pricing department, the contracts department and the billing department, usually at night. Dara takes the paper: anything with a number on it comes through her. When a buyer asks for a price or a contract, the ask lands on her desk with the buyer's exact words, she drafts the proposal from your price sheet and your standard terms, she checks that every document about the deal says the same thing, and she drafts the invoice once the number is confirmed. This guide sets her up on Hermes Agent, a free and open source agent from Nous Research that runs on your own machine.

The software is free. The model behind it is not always free, and the real work is writing down your prices and your terms once, in one place.

**Checked on 2026-11-21.** Commands and limits change. Sources are at the bottom.

**How this guide was made:** every command below was copied from the Hermes Agent documentation on that date. We have not test-run them on a fresh machine. If a step does not behave the way it says, see "If a step breaks" near the end: your AI can patch it for you in a minute.

**Dara drafts. She is not a lawyer.** Nothing she writes is legal advice, and nothing goes to a buyer until you have read it. For a contract that matters, have your own lawyer read it too.

## What Dara does

- Keeps one price sheet and one page of standard terms. Those two files are the only source for a number or a clause.
- Opens one file per deal, starting with the buyer's exact words: what they asked for, in quotes, with the date.
- Drafts the proposal and the contract terms from the sheet and the standard terms, and flags anything the buyer asked for that is not on either.
- Compares every document about a deal (proposal, agreement, onboarding copy, invoice) and tells you where two of them disagree.
- Drafts the invoice only after you confirm the number, and lists what is owed and when so whoever tracks payments can track it.
- Sends you a few lines each weekday morning, only if something changed.
- Does not send a proposal, sign anything, change a price, or take a payment. You do.

## What you need before you start

| You need | Why | Cost | Checked |
|---|---|---|---|
| A computer that stays on: macOS, Linux, Windows or WSL2 | The morning note only runs while Hermes' gateway is running | Yours already | 2026-11-21 |
| Git and curl | The installer uses them | Free | 2026-11-21 |
| Hermes Agent | Runs Dara | Free, MIT licence | 2026-11-21 |
| A model with at least 64,000 tokens of context | Hermes rejects smaller models at startup | Depends on the provider you pick. Not measured here | 2026-11-21 |
| Your real prices and the lowest you will go | She drafts from them, so they have to be written | Yours already | 2026-11-21 |
| Your standard terms: payment, length, cancellation, what you never offer | Same reason | Yours already | 2026-11-21 |
| Telegram (optional) | To get the morning note on your phone | Free | 2026-11-21 |

On a brand new install the setup wizard may offer a "Free, start now" choice that needs no account. The Hermes docs say it is shown only when it is available, so do not plan around it.

## The honest timeline

| When | What | Your time |
|---|---|---|
| Day 1 | Install Hermes and get one normal chat working | The Hermes docs say under 5 minutes. Allow 30 |
| Day 1 | Create Dara and give her a desk folder | 20 minutes |
| Day 1 | Write the price sheet and the standard terms | 1 to 2 hours. This is the part that decides everything |
| Day 2 | Turn on the morning note | 20 minutes |
| Week 1 | Open a deal file for every ask, with the buyer's exact words, and read what she drafts | 10 minutes a deal |

The slow part is not the install. It is deciding your floor and your terms before a buyer is on the phone. Most owners have never written down the lowest price they will take. Until that is on a page, she has nothing to hold the line with.

Three limits to know before you start:

- She only knows a deal exists if a file is opened for it. An ask that stays in a text thread never reaches her.
- She does not read your inbox, your CRM or your payment account in this guide. She works from what you paste in, and she drafts for you to send.
- She can tell you two documents disagree. She cannot tell you which one the buyer agreed to. That is yours to answer.

## Setup, in this order

### 1. Install Hermes and get one chat working

Do not add anything else until a plain conversation works. The Hermes quickstart is blunt about this, and it is right.

On macOS and Windows the Hermes docs recommend the Hermes Desktop installer from https://hermes-agent.nousresearch.com/. For a command line only install on macOS, Linux or WSL2:

```bash
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
```

Reload your shell, pick a model, and start a chat:

```bash
source ~/.zshrc
hermes setup
hermes
```

You should see: a welcome banner naming your model, and a reply to a simple question. If replies are empty or broken, run `hermes model` again and confirm the provider, the model and the login. If anything else looks off, run `hermes doctor`.

### 2. Create Dara

Every agent in Hermes gets its own profile: its own settings, memory and identity. Never point two agents at the same profile.

```bash
hermes profile create dara --description "Deal desk. Drafts proposals, contract terms and invoices from the one price sheet and the standard terms, keeps one file per deal starting with the buyer's exact words, and flags any two documents that disagree. Never sends, signs, changes a price or takes a payment. The owner does."
```

Creating the profile also creates a `dara` command. Check it:

```bash
dara chat
```

You should see: the same kind of chat as step 1, now running as the `dara` profile.

### 3. Give Dara her identity

In Hermes, who an agent is lives in a file called `SOUL.md` inside that agent's home. For a profile named `dara` that is:

```text
~/.hermes/profiles/dara/SOUL.md
```

Hermes creates a starter file there. Replace its contents with the text in "Dara's instructions" below. Keep it about who she is and how she talks. Your prices and your terms go in different files, in the next step.

You should see: after starting a new `dara chat`, she introduces herself as your deal desk when asked who she is.

### 4. Build her desk: the rules, the price sheet, the terms

This is the step everybody rushes, and it is the one that makes Dara useful.

Make a folder for her to work from:

```bash
mkdir -p ~/dara-desk/deals ~/dara-desk/proposals ~/dara-desk/agreements ~/dara-desk/invoices ~/dara-desk/copy
```

Put three files in it. Hermes loads `AGENTS.md` from the folder you start a chat in, and Dara reads the others as she works.

`~/dara-desk/AGENTS.md`, the rules:

```text
# How Dara works this desk

- price_sheet.md and terms.md are the only source for a number or a clause. Never use one from memory, from an old deal, or from a guess.
- deals/ holds one file per deal, named for the buyer. Each file starts with the ask: the date, who asked, and the buyer's exact words in quotes. Never paraphrase the buyer. If I give you a summary, ask me for their words.
- Under the ask, keep the deal's state in plain lines: what was quoted, what was agreed, what is signed, what is invoiced, what is paid, and the date of each.
- When a deal needs a proposal, draft it in proposals/ from price_sheet.md and terms.md. List at the top anything the buyer asked for that is not on either file.
- Anything below the lowest price allowed, or any term on the "Never offered" list, is a stop. Say so, say who can approve it, and wait for me.
- Before anything goes to a buyer, compare every document about that deal: the proposal, the agreement in agreements/, any onboarding or welcome copy in copy/, and the invoice. Tell me every place two of them disagree, with both versions side by side.
- Draft an invoice in invoices/ only after I confirm the number. Put the amount, the due date and the terms on it, and add a line to the deal file so payments can be tracked.
- You draft. You never send a proposal, sign anything, change the price sheet or the terms, or take a payment. Show it to me.
- You are not a lawyer. If a clause is unusual or I ask what something means legally, say that it needs a lawyer.
- If something is not in price_sheet.md or terms.md, say so and ask me once. Never fill a gap with a guess.
```

`~/dara-desk/price_sheet.md`, the one price sheet:

```text
# Price sheet

Last changed: {date}, by {name}

| Item | Price | Unit (one time, monthly, per user) | Lowest price allowed | Who can approve less |
|---|---|---|---|---|
| {item} | {price} | {unit} | {floor} | {name} |

## Partner and referral deals
| Partner | What they sell | Their share | Paid when |
|---|---|---|---|
| {partner} | {item} | {share} | {on signature, on payment} |
```

`~/dara-desk/terms.md`, the standard terms:

```text
# Standard terms

Last changed: {date}, by {name}

## Every deal
- Payment: {due on signature, net 15, monthly in advance}
- Length: {month to month, 12 months}
- Cancellation: {notice period and how}
- A quote is good for: {days}
- How we sign: {the tool or the method}

## Never offered
- {discounts, guarantees, exclusivity or terms nobody may promise}

## Needs my approval every time
- {anything below the floor, custom terms, a new partner split}
```

You should see: when you run `dara chat` from inside `~/dara-desk` and ask what a product costs or what your payment terms are, she answers from the two files or says it is not written.

### 5. Fill in the sheet and the terms

Run `dara chat` from inside `~/dara-desk` and send her this:

```text
Help me fill in price_sheet.md and terms.md. Ask me for one item at a time: its price, its unit, the lowest price I allow and who can approve less. Then ask me for my payment terms, deal length, cancellation, how long a quote is good for, how we sign, what I never offer and what needs my approval every time. Write my answers into the two files exactly as I give them. If I do not know something, write "unknown" and move on.
```

You should see: a price sheet with every item you sell and a floor for each, and a terms page in your own words.

### 6. Schedule the morning note

Hermes has a built-in scheduler. Give the job Dara's folder so she reads your deals when she runs. The folder must be written as a full path.

```bash
hermes -p dara cron create "weekdays at 8am" \
  "Morning note. Read every file in deals/. Report only what changed since yesterday or needs me today: a new ask with no proposal yet, a proposal waiting on my approval, a deal agreed but not signed, a signed deal with no invoice, an invoice past its due date, or two documents that disagree. One line each, with the buyer's name. If nothing changed, reply with the single line: Nothing changed. Send nothing to anyone else and change no file. Keep it under 120 words." \
  --workdir /Users/you/dara-desk \
  --name "Deal desk note"
```

Schedules run from a gateway, and each profile has its own. Start Dara's, and check that her scheduler is alive:

```bash
dara gateway start
hermes -p dara cron status
```

`dara gateway install` keeps it running after a restart.

By default the note is saved on your machine. To get it on your phone, connect Telegram first with `hermes gateway setup`, then create the job with `--deliver telegram` added.

One thing to check on your machine: whether your times come out in your own timezone. Hermes reads the schedule in the profile's `timezone:` setting. Add `--timezone America/Los_Angeles` (or your zone) to pin it.

You should see: a short deal desk note waiting for you the next weekday morning, or the one line saying nothing changed.

### 7. Run your first deal, by hand

Take one real ask from a buyer this week. From inside `~/dara-desk` run `dara chat` and send her this, with the buyer's message pasted in exactly as they wrote it:

```text
New ask. Buyer: {name}. Date: {date}. Their exact words: "{paste the buyer's message}". Open a deal file for them in deals/ starting with those words in quotes. Then draft a proposal in proposals/ from price_sheet.md and terms.md. At the top of the proposal list anything they asked for that is not on the sheet or in the terms, and anything that would go below a floor. Do not send anything.
```

You should see: a new file in `deals/` that opens with the buyer's words in quotes, and a draft proposal where every price and term traces to one of your two files, with the gaps listed at the top.

## Dara's instructions

Paste this into `~/.hermes/profiles/dara/SOUL.md`. Change the parts in braces.

```text
# Who you are

You are Dara. You are the deal desk for {your business}.

Your job is to get every deal in writing and keep it right: the price comes from the price sheet, the terms come from the standard terms, the buyer's own words are on file, and every document about a deal says the same thing.

# How you work

- Anything with a number on it comes through you: pricing, terms, proposals, invoices.
- You start every deal from the buyer's exact words, in quotes. You never paraphrase a buyer.
- You draft from the price sheet and the standard terms. A number or a clause that is not on them does not exist until your owner writes it.
- You hold the floor. Below it, or on the never offered list, you stop and say who can approve it.
- You compare the documents. When two disagree you show both versions side by side and ask which one stands.
- You draft the invoice only after the number is confirmed.
- You never send, sign, change a price or take a payment. Your owner does.
- You are not a lawyer and you say so when a question is a legal one.

# How you talk

- Cool, firm and unhurried.
- Short sentences. The number first, then the reason.
- One line per open item: the buyer, what is waiting, on whom.
- When nothing changed, you say so in one line and stop.
- When you ask a question, ask exactly one.
```

This persona was written from Dara's job description for this guide. It is not a copy of the instructions she runs on in our own team.

## First test

Run `dara chat` from inside `~/dara-desk` and send her this:

```text
List every item on price_sheet.md with its price and its lowest price allowed. Then tell me which rows are missing a price, a unit or a floor. Then read terms.md and tell me which of these are not written yet: payment, length, cancellation, how long a quote is good for, how we sign, what is never offered. Use only what is in the two files. Change nothing.
```

A good answer repeats your prices and terms exactly as written and names the gaps instead of filling them.

If she states a price or a term that is not in the files, tighten the first rule in `AGENTS.md` and ask her where it came from.

## When it goes wrong

| What you see | Why | Fix |
|---|---|---|
| Empty or broken replies | Model or login is wrong | `hermes model`, then a plain chat before anything else |
| She does not know your prices or terms | The chat was not started in `~/dara-desk` | Start `dara chat` from that folder |
| A proposal has a price that is not on the sheet | She guessed, or the sheet is out of date | Fix the sheet first, then tighten the "only source" rule in `AGENTS.md` and `SOUL.md` |
| She paraphrased the buyer | You gave her a summary and she kept it | Paste the buyer's own message and tell her to replace the summary with it in quotes |
| She missed a mismatch | One of the documents was not in her folder | Put the agreement in `agreements/` and the onboarding copy in `copy/`, then ask again |
| No morning note | Dara's gateway is not running, or the job is missing its folder | `hermes -p dara cron status`, `dara gateway start`, and recreate the job with `--workdir` |
| The note arrives at the wrong hour | The schedule is running in another timezone | Recreate the job with `--timezone` set to your zone |
| Anything else | | `hermes doctor` |

## If a step breaks

Hermes changes fast, so a command here can drift from what your version expects. You do not need us to fix it. Paste this into Hermes itself, or into Claude or ChatGPT, with the error underneath:

```text
I am following a setup guide for a deal desk agent on Hermes Agent. This step failed. Here is the step, the exact command I ran, and the full error. Check the current Hermes Agent docs at https://hermes-agent.nousresearch.com/docs, tell me what changed, and give me the corrected command.
```

`hermes doctor` is worth running first. It catches most setup problems on its own.

## What it costs to run

- Hermes Agent: free, MIT licence.
- The model: billed by whichever provider you choose in `hermes model`. Every morning note and every draft is model usage. Not measured here, so no number is given.
- Your signing and payment tools bill separately. This guide does not connect to any of them.

## Want it set up for you?

CAMBY Ai sets Dara up and runs her for you: https://mikecamby.com

## Credit

Dara runs on [Hermes Agent](https://github.com/NousResearch/hermes-agent), built by Nous Research and released under the MIT licence. Hermes is not ours. This guide is a walkthrough for using it as a deal desk. Not affiliated with or endorsed by Nous Research or Telegram. Nothing in this guide is legal advice.

## Sources

All read on 2026-11-21 from the Hermes Agent repository and its documentation.

- https://github.com/NousResearch/hermes-agent : install command, first-run commands, licence.
- https://hermes-agent.nousresearch.com/docs/getting-started/quickstart : install paths, `hermes setup`, the 64,000 token minimum, the "Free, start now" option, `hermes doctor`.
- https://hermes-agent.nousresearch.com/docs/user-guide/profiles : `hermes profile create`, `--description`, one profile per agent, the per-profile command.
- https://hermes-agent.nousresearch.com/docs/user-guide/features/personality : `SOUL.md`, where it lives and what belongs in it.
- https://hermes-agent.nousresearch.com/docs/user-guide/which-file-does-what : `SOUL.md` versus `AGENTS.md`.
- https://hermes-agent.nousresearch.com/docs/user-guide/features/cron : `cron create`, "weekdays at", `--workdir`, `--deliver`, `--name`, `--timezone`.
- https://hermes-agent.nousresearch.com/docs/user-guide/messaging : `hermes gateway setup`, `start`, `install`, `status`.
- https://hermes-agent.nousresearch.com/docs/user-guide/multi-profile-gateways : one gateway per profile, `<profile> gateway start`, `hermes -p <profile> cron status`.
