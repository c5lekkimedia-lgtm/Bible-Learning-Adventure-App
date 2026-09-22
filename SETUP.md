# Bible Adventure — Firebase Setup Guide

This turns the app into something every tablet, phone and computer shares:
one class roster, one set of lessons, and progress that's visible from any
device. Everything below fits inside Firebase's free "Spark" plan for a
Sunday School's worth of children — there is no cost to set this up or run
it at this scale.

You'll need about 20 minutes and a Google account.

---

## 1. Create the Firebase project

1. Go to <https://console.firebase.google.com> and sign in.
2. Click **Add project**, give it a name (e.g. "covenant-nation-lekki-sunday-school"), and finish the wizard. You can decline Google Analytics — it isn't needed.

## 2. Turn on Firestore (the database)

1. In the left sidebar, click **Build → Firestore Database**.
2. Click **Create database**.
3. Choose **Start in production mode** (not test mode), pick the location closest to your church, and click **Enable**.
4. Once it's created, click the **Rules** tab.
5. Delete the default contents and paste in everything from the `firestore.rules` file delivered alongside this guide.
6. Click **Publish**.

## 3. Turn on Authentication (teacher login)

1. In the left sidebar, click **Build → Authentication**.
2. Click **Get started**.
3. Under **Sign-in method**, enable **Email/Password** and save.
4. Go to the **Users** tab and click **Add user**. Enter the email and password your teacher will sign in with (e.g. `teacher@yourchurch.org`). This is the account used to log in to the Teacher area in the app — there's no separate in-app passcode any more.
   - Add one user per teacher who needs to publish lessons. There's no limit on the free plan.

## 4. Connect the app to your project

1. In the Firebase console, click the gear icon next to **Project Overview → Project settings**.
2. Scroll to **Your apps** and click the **</>** (web) icon to register a new web app. Give it any nickname — you don't need Firebase Hosting checked at this step.
3. Firebase will show a code snippet with a `firebaseConfig` object like:
   ```js
   const firebaseConfig = {
     apiKey: "AIza...",
     authDomain: "your-project.firebaseapp.com",
     projectId: "your-project",
     storageBucket: "your-project.appspot.com",
     messagingSenderId: "...",
     appId: "..."
   };
   ```
4. Open `bible-adventure-firebase.html` in a text editor, find the `firebaseConfig` object near the top of the `<script type="module">` block, and replace the placeholder values with the ones Firebase gave you.
5. Save the file.

## 5. Put the app somewhere everyone can open it

You have two easy options:

**Option A — Firebase Hosting (recommended, free, gives you a stable link)**
1. Install Node.js if you don't already have it, then install the Firebase CLI:
   ```
   npm install -g firebase-tools
   ```
2. In a terminal, in the folder containing `bible-adventure-firebase.html`:
   ```
   firebase login
   firebase init hosting
   ```
   - Choose your project when asked.
   - When asked for the public directory, create a folder called `public` and rename `bible-adventure-firebase.html` to `index.html` inside it.
   - Answer "No" to configuring as a single-page app, and "No" to setting up automatic builds.
3. Deploy:
   ```
   firebase deploy
   ```
4. Firebase will print a URL like `https://your-project.web.app` — that's the link every device uses.

**Option B — Any other static host**
Netlify, GitHub Pages, or your church's existing web hosting will all work, since the app talks to Firebase over the internet regardless of where the HTML file itself is hosted. Just upload the single HTML file (renamed to `index.html` if your host expects that).

Whichever you choose:
1. Back in the Firebase console, go to **Authentication → Settings → Authorized domains**.
2. Add the domain the app is hosted on (e.g. `your-project.web.app`, or your Netlify/GitHub Pages domain). Without this step, teacher sign-in will be blocked.

## 6. Load the starter lessons

1. Open the app and sign in as a teacher (Teacher tab → the email/password you created in step 3).
2. Go to **Lessons** and click **Load the 4 starter lessons** — this copies in the four lessons that shipped with the original prototype so you don't have to retype them. It's safe to click any time; it only ever adds lessons, never removes or overwrites existing ones.
3. From there, use **New lesson** to add future weeks, including a picture for each one.

## What this gives you

- **Shared class roster** — a child's name, age group and points are the same no matter which tablet or computer they use.
- **Optional per-child PIN** — a child can set a 4-digit PIN when their profile is created, so another child on the same tablet can't tap into their page. A teacher can rename a child, remove them, or reset a forgotten PIN from the Class tab.
- **Live lesson publishing** — the moment a teacher publishes a lesson, every open device updates automatically (no refresh needed).
- **Real teacher accounts** — sign-in is handled by Firebase Authentication, not a shared passcode.
- **Cost** — free at this scale. Firebase's free tier allows roughly 50,000 document reads and 20,000 writes per day and 1 GB of storage; a single class doing this weekly would use a small fraction of that even after years of use.

## Things worth knowing

- **The child PIN is a deterrent, not real security.** It's stored as a SHA-256 hash (never the plain digits), so nothing readable sits in the database — but the `profiles` collection is still openly readable and writable by design, since children don't log in. A technically determined adult could still get at the data directly; a child tapping around cannot. This is a reasonable trade-off for a small, trusted Sunday School setting — just don't post the app's link somewhere public.
- **Lesson images live inside the lesson record itself** (as compressed photos, typically 100–300 KB each), which keeps things simple but means a single lesson can't exceed Firestore's 1 MB document limit — the app warns you if a photo pushes a lesson over that.
- **The AI "draft a quiz" button is gone.** It relied on tools that only exist inside Claude's own hosting. To draft quiz questions, open a separate Claude.ai tab, ask it to write age-appropriate multiple-choice questions from your lesson summary in the JSON format the app expects, and paste the result into the Quiz field.
- **Adding more teachers** is done from the Firebase console (Authentication → Users), not inside the app, to avoid signing the current teacher out.
