# TECHVENTURE Live Quiz

A live team quiz for your event. One login per team, an admin control room, a timer on every
question, a start that happens at the same moment on every device, and a leaderboard that rewards
fast, correct answers. No database and no `npm install` — it is a single Node.js file.

## Run it (2 minutes)

1. Install Node.js 18 or newer (https://nodejs.org).
2. In this folder, copy `.env.example` to `.env` and set `ADMIN_PASSWORD` to a long passphrase.
3. Run `node server.js`
4. Open `http://localhost:3000/` and sign in as `admin` with that password.

| Page | Who | What |
|------|-----|------|
| `/` | teams and admin | login; teams play here |
| `/admin` | organisers | create team logins, upload quiz, start/skip/end, settings, live leaderboard |
| `/board` | projector | big-screen question, timer, answer spread, leaderboard, podium |

## Event flow

1. **Teams tab** → type one team name per line → *Create logins*. A table of usernames and random
   passwords appears **once**. Copy it or download the CSV and hand each team its login.
   (A lost password can be replaced with *New password*.)
2. **Quiz tab** → upload `.json` or `.csv` (samples are linked there), or add questions by hand.
   Every question has its own timer (5–300 s) and max points.
3. Open `/board` on the projector. Teams sign in on their phones/laptops.
4. **Live control** → *Start quiz for all teams*. A countdown (default 10 s) runs against the
   server clock, so every team's first question opens at the same instant.
5. Questions advance automatically (turn this off in Settings to advance manually). After each
   question the answer, how many teams chose each option, and the leaderboard are shown.

## Scoring

```
correct:  points = maxPoints × (0.3 + 0.7 × timeLeft / timeLimit)
wrong or no answer: 0
```

Answer instantly and you get ~100% of the points; answer at the buzzer and you still get 30%.
Ties are broken by number of correct answers, then total response time. Change `MIN_FRACTION`
at the top of `server.js` to make late answers worth more or less.

## Quiz file formats

JSON (`answer` is `"A"`–`"D"`):

```json
{ "questions": [
  { "question": "Which data structure is LIFO?", "options": ["Queue","Stack","Array","Tree"], "answer": "B", "time": 15, "points": 1000 }
] }
```

CSV (header row optional): `question,option A,option B,option C,option D,answer,time,points`
Two to four options are allowed (leave C and D empty for True/False).

## Security: what is built in

- Passwords stored only as salted **scrypt** hashes; team passwords are random 10-character strings.
- Signed session cookies: `HttpOnly`, `SameSite=Strict`, `Secure` over HTTPS.
- **One device per team**: signing in again signs the old device out.
- Login rate limiting per username and per IP; same-origin check on every POST.
- All timing and scoring runs on the server. Browsers only display the clock.
- Correct answers and the leaderboard are never sent to teams while a question is live.
- Strict Content-Security-Policy, no inline scripts, all text rendered via `textContent`, request size limits.
- Data (teams, scores, questions) is saved to `data/techventure.json` and survives restarts.

## Before the real event

- **Use HTTPS if anyone connects over the internet.** On a college Wi-Fi with plain
  `http://<your-ip>:3000` the passwords travel unencrypted. Either deploy to a host that gives you
  HTTPS (Render, Railway, Fly.io) and set `FORCE_HTTPS=1`, or put the server behind a tunnel such as
  Cloudflare Tunnel.
- **Run exactly one copy** of the server (game state lives in that process).
- Hosting on a free tier that sleeps will interrupt the quiz — use a paid/always-on instance, or run it on a laptop on the venue network (allow Node through the firewall, share `http://<laptop-ip>:3000`).
- Do a full rehearsal with a few phones: create test teams, run the sample quiz, then *Reset*.
- Do not share your `.env` or the `data/` folder.

## Files

```
server.js            server, auth, game engine, live updates
public/              pages (index, admin, board), css, js, sample quizzes
.env.example         settings template
```
