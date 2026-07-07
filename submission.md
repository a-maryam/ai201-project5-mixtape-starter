# Mixtape Bug Hunt — Submission

## AI Usage

> Describe specifically how you used AI tools during this project. Be honest about
> the collaboration — this section is not meant to prove you did everything alone.

- **Codebase navigation:** What did you ask AI to explain, summarize, or trace during Milestone 1?
I asked AI to summarize each service file and trace the flow of one endpoint.
- **Debugging:** What did you ask AI to help you understand about suspicious functions or code paths?
- **Where AI helped:** AI was useful to develop a first impression of the codebase.
- **Where AI was wrong, incomplete, or pointed you the wrong way, and how you caught it:** ...

---

## Codebase Map

> Write this *before* touching any issue. Name the responsibility of each major
> file/module — don't just list file names.

### Main files and their roles

- `seed_data.py` — populates the database
- `app.py` / `app:create_app` — Flask app factory, sets up SQLAlchemy, registers blueprints (feature/pattern of Flask which organizes routes into components before attached to app)
- `models.py` — contains data models including User, Song, ListeningEvent, Rating, Playlist, Notification
- `routes/` — controller layer -- songs.py: searches and gets songs from database. playlists.py: create, get playlist and list/add songs, users.py: get user, streak, notifications, feed.py: friends listening now, activity feed
- `services/` — streak_service.py increments the streak on consecutive days otherwise it resets. feed_service.py gets friends listening info and get activity feed is a general activity log method. search_service.py is for searching songs by title/artist that the user likes. notification_service.py handles creating and getting notifications. playlist_service.py creates and gets playlists.

### Data flow: How is song added to user's feed?

> Trace it step by step: which route is hit, which service functions are called,
> in what order, what gets written to the database, etc.

1. POST /songs/<song_id>/listen is hit and calls record_listening_event(user_id, song_id)
2. Hit services/streak_service.py: load the user and create ListeningEvent. update_listening_streak is called
3. update_listening_streak in streak_service.py, streak is calculated
4. Feed is read with routes/feed.py, GET /feed/ endpoints (multiple) are used to read. These endpoints are hit when you look at the listening now and activity tab of the app.

### Patterns noticed

- Routes do not interact with the database directly or raise errors, they convert the errors to status codes. 

---

## Root Cause Analysis Entries

> One complete entry per bug fixed (minimum 3, stretch: 4th and 5th). All five
> fields are required for each entry. Fill these in as you go — not at the end.

### Bug #3: the same song keeps showing up twice in search 

**How you reproduced it**
- Steps / inputs / sequence of actions / data condition that triggered the behavior:
- I searched for songs with multiple tags like "Harlem Renaissance" which has three tags: rap, hip-hop, and soul. So when we hit that endpoint, we have Harlem Renaissance appear 3 times. 

**How you found the root cause**
- Files examined, in order: we go to search_service.py and look at the search function. 
- Navigation path (what led you from symptom to cause): This sort of issue seems like an SQL query related thing (from my experience) and we see a join (outer join) happening in search_songs. 
- The moment you were confident you'd found the *actual* cause (not just a suspicious area): I was pretty sure on first look because I have done joins before and realized that I needed to filter out duplicates.

**The root cause**
- Plain-English explanation of exactly what was wrong. Name the specific
  condition, comparison, or missing step — not "there was a bug in X logic."

**Your fix and side-effect check**
- What you changed and why it resolves the root cause: I removed the join and just filtered songs by srtist and title. 
- Related functionality you checked afterward to confirm nothing else broke: I ran the test_search tests and everything passed. 
- Commit: `fix: ...`

---

### Bug #1: My listening streak keeps resetting 

**How you reproduced it**
- I set the user last listened at to Saturday, and then I called update_listening_streak for the next day (Sunday). The streak did not update to 2 as it should have.

**How you found the root cause**
-

**The root cause**
-

**Your fix and side-effect check**
-

---

### Bug #4: I got notified when a friend added my song to a playlist but not when they rated it

**How you reproduced it**
- I rated the song as another user in relation to the sharer. Then I checked the sharing user's notification, and the song_rated notification was missing.

**How you found the root cause**
-

**The root cause**
-

**Your fix and side-effect check**
-

---

<!-- Stretch: duplicate the block above for a 4th and/or 5th bug fix -->

---

## Commit Log

> Paste (or reference) your `git log --oneline` screenshot here, showing one
> `fix:` commit per bug on the `bugfix/mixtape` branch.

```
[screenshot or pasted output of: git log --oneline]
```