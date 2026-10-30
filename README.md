# Wednesday Hoops sign-up sheet

Single-page site (`index.html`) hosted on GitHub Pages, with Firebase Firestore as the shared database.

## 1. Create the database (free, ~5 min)
1. Go to https://console.firebase.google.com and click **Add project** (skip Google Analytics).
2. In the project: **Build → Firestore Database → Create database**. Pick any region, start in **production mode**.
3. Open the **Rules** tab, paste the contents below, and click **Publish**:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /signups/{id} {
      allow read: if true;
      allow create: if request.resource.data.name is string
                    && request.resource.data.name.size() > 0
                    && request.resource.data.name.size() <= 40
                    && request.resource.data.runDate is string;
      allow update: if request.resource.data.diff(resource.data).affectedKeys().hasOnly(['attended']);
      allow delete: if true;
    }
  }
}
```

4. **Project settings (gear icon) → General → Your apps → Web (`</>`)**. Register an app and copy the `firebaseConfig` values.
5. Paste those values into the `firebaseConfig` block near the bottom of `index.html`.

## 2. Host on GitHub Pages
1. Create a new public repo and upload `index.html` and this README.
2. **Settings → Pages → Build and deployment**: Source = *Deploy from a branch*, Branch = `main` / `(root)`, Save.
3. After a minute your site is live at `https://<your-username>.github.io/<repo-name>/`.

## How it behaves
- Always shows the **next Wednesday** (today, if it's Wednesday). Thursday 11/5 rolls over to 11/11.
- Anyone can sign up, tick "Showed up", or remove a name. No logins.
- Old weeks stay in the database. The "Times showed up" section is the all-time attendance tally.

## Notes
- The Firebase config in `index.html` is safe to be public. The rules above are what control access.
- Anyone with the link can edit, which is what you asked for. If a troll shows up, you can delete entries in the Firebase console.
