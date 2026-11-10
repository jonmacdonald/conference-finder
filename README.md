# Conference Finder

A Claude skill that finds the conferences worth speaking at, sponsoring or attending, starting from the companies you want to reach.

Created by [Jon MacDonald](https://jonmacdonald.com) · [github.com/jonmacdonald/conference-finder](https://github.com/jonmacdonald/conference-finder)

**In this repository**
- `conference-finder.zip` is the file you upload to Claude.
- `conference-finder/SKILL.md` is the same skill, unzipped, so you can read it before installing.
- `LICENSE` is the Creative Commons Attribution 4.0 International license (CC BY 4.0).
- This README is the setup guide.

## How it works

Conference Finder is a Claude skill that builds a ranked list of conferences worth speaking at, sponsoring or attending. It starts with the companies you want to reach and finds the events their people actually show up at. It then adds events that match the kinds of conferences you describe.

For the best events it also finds:
- the official website and who runs it
- whether the event is taking speaker submissions, the deadline, and a direct link to the form
- the person who coordinates speakers, with their email and LinkedIn
- who spoke at past editions
- which of those past speakers you already know, and who to approach first

Everything goes into a Google Sheet, sorted by event date.

It only produces a list. It never contacts anyone, submits anything, or writes to your CRM.

---

## 1. What you need before you start

**Claude**
- A Claude plan that supports skills (Free, Pro, Max, Ultra, Team or Enterprise). On Team and Enterprise, an owner can turn off skill creation.
- Code execution turned on. On Free, Pro, Max and Ultra, turn on "Code execution and file creation" under Settings > Capabilities. On Team and Enterprise an owner turns it on in Organization settings > Plugins & skills, on the Policy tab.
- Web search turned on.

**Connectors**

| Connector | Required? | What it's for | How to add it | Cost |
|---|---|---|---|---|
| **ConferenceGrid** | Required | Event data: dates, sponsors, exhibitors, speakers, calls for speakers | Add a custom connector with the URL `https://conferencegrid.com/api/mcp` | Free, no API key, read-only |
| **Google Workspace** | Required | Writes the results sheet, and reads your target list if it lives in a sheet | Add it from the connectors directory and sign in with the Google account that should own the sheet | Free |
| **Apify** | Full runs only | Mines LinkedIn posts for events people attend but don't speak at, and reads event sites that need a browser | Add a custom connector with the URL `https://mcp.apify.com` and sign in | Paid Apify account. The skill stops and asks before spending more than its budget cap ($50 by default) |
| **Apollo.io** | Optional | Finds decision-makers at your target companies, and coordinator emails when an event doesn't publish them | Add a custom connector with the URL `https://mcp.apollo.io` and sign in | Uses Apollo credits. Capped at 200 per run by default, and it asks before going over |
| **HubSpot** | Optional | Finds people you already know among past speakers, for warm intros | Add it from the connectors directory | Free, read-only use |
| **Perplexity** | Optional | Extra web research | Add it from the connectors directory | Your Perplexity plan |

With no CRM connected, you can give the skill a LinkedIn connections export (CSV) for warm-intro matching, or skip that part.

---

## 2. Install the skill

1. [Download `conference-finder.zip`](https://github.com/jonmacdonald/conference-finder/raw/main/conference-finder.zip) and keep it as it is. The skill folder is inside it. (GitHub's green **Code > Download ZIP** button gives you the whole repository instead, which Claude won't accept as a skill.)
2. In Claude, go to **Customize > Skills**.
3. Click **+**, choose **+ Create skill**, then **Upload a skill**.
4. Select the zip file. The skill appears in your list and you can turn it on or off.

On Team and Enterprise plans, an uploaded skill is private until you share it.

---

## 3. Set it up

Everything specific to you lives in **Section 1 (Configuration)** at the top of `SKILL.md`. It ships blank. You can fill it in two ways.

### Option A: let the skill interview you (recommended)

Ask Claude to "run the conference finder". When Section 1 is blank, the skill asks you a few questions, one at a time, before it starts:
- who the list is for
- what you want from events
- what kinds of conferences to look for
- who you want to meet
- where your target company list is
- which Google account to use
- how to find people you already know

It runs with your answers, then shows you a filled-in Section 1 to keep.

A skill can't edit its own file. To keep your answers for later runs:
1. Paste the block it gives you into `SKILL.md`.
2. Zip the folder again.
3. Upload it again.

Scheduled runs need this step, because they won't stop to interview anyone.

### Option B: fill in Section 1 yourself

Open `SKILL.md`, edit the YAML block in Section 1, then zip and upload.

**What you need to add, and what the skill handles**

| Setting | Needed? | What to put | Who fills it |
|---|---|---|---|
| `speaker_name`, `speaker_company`, `speaker_role` | Required | The person the list is for | You |
| `google_account` | Required | The Google account the sheet should belong to, e.g. `you@yourco.com` | You |
| `event_goals` | Required | Any of `speak`, `sponsor`, `attend` | You |
| `event_focus` | Required | One sentence on the conferences you want (see section 4) | You |
| `search_topics` | Required | 5–12 short search terms, one or two words each | You, or the skill suggests them from `event_focus` |
| `keynote_topics` | Required if speaking | The talks you give, for judging fit | You |
| `target_titles` | Required | Job titles of the people you want in the room | You |
| Target company list: `seed_companies`, or `seed_spreadsheet_id` plus `seed_tab_name` | Required | Type the companies in, or point to a Google Sheet. The sheet ID is the long string in its URL between `/d/` and `/edit` | You. The skill can also propose a list from your description for you to approve |
| `icp_description`, `revenue_floor_usd`, `revenue_ceiling_usd` | Optional | The kind of company you want to reach | You |
| `exclude_topics`, `regions`, `search_cities`, `formats`, `min_attendance` | Optional | Filters (see section 4) | You |
| `past_speaking_events` | Optional | Events you've spoken at before. They score higher | You |
| `support_person` | Optional | Whoever handles outreach for you | You |
| `network_source`, `connections_csv_path` | Optional | `hubspot`, `csv` or `none` | You |
| `output_spreadsheet_id` | Leave blank | The skill creates the sheet on the first run and gives you its ID to paste in, so later runs update the same sheet | The skill |
| Everything under "Deep research", "Run limits" and "Scoring weights" | Optional | Sensible defaults are already set | Change only if you want to |

---

## 4. Tell it what kinds of conferences you want

These settings decide which events it looks for and how it ranks them:

- **`event_goals`** changes what it checks. With `speak`, it looks for calls for speakers and scores open ones higher. With `sponsor`, it checks sponsorship availability. With `attend`, it focuses on who will be in the room.
- **`event_focus`** is a plain-language description of the events you want. It decides what gets kept and how "topic fit" is scored.
- **`search_topics`** are the short terms used to search the conference database. Keep each to one or two words. Long phrases match nothing.
- **`exclude_topics`, `regions`, `formats`, `min_attendance`** filter out what you don't want.
- **`target_titles`** and your target company list decide which events count as "where your buyers are". This is the strongest signal in the ranking.

**Three example setups**

A B2B SaaS founder who wants speaking slots:
```yaml
event_goals: [speak]
event_focus: "B2B SaaS product, growth and UX leadership conferences"
search_topics: [product, product management, growth, SaaS, UX, product marketing]
target_titles: [VP of Product, Head of Growth, Chief Product Officer]
```

A fintech company picking events to sponsor:
```yaml
event_goals: [sponsor, attend]
event_focus: "Payments, banking and fintech conferences with bank and credit union buyers"
search_topics: [fintech, payments, banking, credit unions, open banking]
exclude_topics: [crypto]
regions: [North America]
target_titles: [Chief Digital Officer, Head of Payments, VP of Innovation]
```

An HR tech seller choosing events to attend:
```yaml
event_goals: [attend]
event_focus: "HR, people operations and talent leadership conferences"
search_topics: [HR, HR technology, talent, people operations, recruiting]
formats: [in person]
min_attendance: 500
target_titles: [Chief People Officer, VP of HR, Head of Talent Acquisition]
```

---

## 5. Run it

Ask in plain words:
- "Run the conference finder."
- "Find conferences where our target accounts are speaking or sponsoring."
- "Refresh the conference sheet."
- "Which events on the list have open calls for speakers?"

It asks which kind of run you want:

| Run type | What it does | Time and cost |
|---|---|---|
| **Full run** | Everything, including LinkedIn post-mining through Apify | Slowest. Uses Apify and Apollo within their caps |
| **Quick run** | Conference database and web research, no post-mining | Faster. May use a few Apollo credits for coordinator emails |
| **Refresh** | Re-checks events already in the sheet: dates, speaker-call status, sponsors | Fast. Adds no new events |
| **Dry run** | Shows what it would search and what it would cost, and writes nothing | Free |

A first full run over 50–150 target companies can take an hour or more. Scheduled runs always use Quick run, and only work once Section 1 is filled in.

---

## 6. What you get

A Google Sheet with three tabs:
- **Events**: every conference found, with a score, a tier (A, B or C) and the evidence behind it. Tier A and B events also get the full research: website, organizer, the organizer's other events on the sheet, submission status, deadline and form link, coordinator with email and LinkedIn, past speakers, past speakers you know, and a recommended 1:1 approach. Rows are sorted by date, soonest first.
- **People**: decision-makers at your target companies tied to those events.
- **Warm Routes**: people you already know who can introduce you to an event.

You also get a short summary. It covers the top events, any deadlines in the next 30 days, organizers who run several events on your list (one relationship covers all of them), what it couldn't find, and credits spent.

**Columns you can use freely:** Status and Notes are yours. The skill never overwrites them. You can also add columns of your own anywhere, because the skill finds its columns by header name.

---

## 7. How ranking works

Each event gets points:
- +3 for each target company that sponsors, exhibits or sends a speaker
- +2 for each person with a target title tied to it
- +10 if the call for speakers is open (speaking goal only)
- +5 if it's taking sponsors (sponsoring goal only)
- +5 if it's within the next 6 months
- +15 if you've spoken there before
- 0–5 for topic fit, with a one-line reason

The top 15 are Tier A and the next 30 are Tier B. You can change all of these weights in Section 1.

---

## 8. Good to know

- **Conference data runs about a year ahead.** Events further out often haven't published details yet. An empty result means "not published yet", not "nothing there".
- **The event's own site wins.** Where the database and the event site disagree on dates, line-ups or speaker calls, the skill goes with the site and notes the difference.
- **Some events don't take outside speakers.** Some are invitation only, some limit sessions to brand-side speakers, and some bar vendors or service providers. The skill records these rules so you don't pitch an event that can't book you.
- **Emails are never guessed.** An email is either published by the event or verified by Apollo. Otherwise the cell says "Not found".
- **Few warm matches is normal** when your network and the conference's speakers come from different industries. The skill still names one past speaker to approach cold, or tells you to go straight to the coordinator.
- **Before you share your copy with someone else,** clear Section 1 back to blanks. By then it holds your sheet IDs, account and target list.

---

## License and credit

Conference Finder © 2026 Jon MacDonald, released under [Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/). The full license text is in [LICENSE](LICENSE).

You're free to use, change and share it, including commercially, as long as you give credit. When you share it, or anything based on it, credit it like this and say if you changed it:

> Conference Finder by Jon MacDonald ([jonmacdonald.com](https://jonmacdonald.com)), from https://github.com/jonmacdonald/conference-finder, licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).
