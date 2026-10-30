# Wednesday Hoops sign-up sheet

Single-page site (`index.html`) hosted on GitHub Pages, with Firebase Firestore as the shared database.
Anyone can sign up (name + email). Only the admin (you) can remove names, mark attendance, and see emails.

## 1. Create the database (free)
1. Go to https://console.firebase.google.com and click **Add project** (skip Google Analytics).
2. **Build > Firestore Database > Create database**. Pick any region, start in **production mode**.
3. **Project settings (gear icon) > General > Your apps > Web (`</>`)**. Register an app and copy the `firebaseConfig` values into the `firebaseConfig` block in `index.html`.

## 2. Turn on admin sign-in
1. **Build > Authentication > Get started > Sign-in method > Google > Enable** (pick a support email, Save).
2. **Authentication > Settings > Authorized domains > Add domain**: add `<your-username>.github.io`.
3. In `index.html`, set `ADMIN_EMAIL` to your Google email.

## 3. Lock it down with security rules
Firestore > **Rules** tab. Paste this, **replace `YOUR_EMAIL@gmail.com` with your Google email**, and click **Publish**:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    function isAdmin() {
      return request.auth != null
          && request.auth.token.email == 'YOUR_EMAIL@gmail.com'
          && request.auth.token.email_verified == true;
    }

    // Public sign-up list: anyone can read and add; only admin can edit/delete
    match /signups/{id} {
      allow read: if true;
      allow create: if request.resource.data.name is string
                    && request.resource.data.name.size() > 0
                    && request.resource.data.name.size() <= 40
                    && request.resource.data.runDate is string
                    && request.resource.data.attended == false;
      allow update, delete: if isAdmin();
    }

    // Private emails: anyone can add, only admin can read/edit/delete
    match /contacts/{id} {
      allow create: if request.resource.data.email is string
                    && request.resource.data.email.size() > 3
                    && request.resource.data.email.size() <= 100
                    && request.resource.data.name is string
                    && request.resource.data.name.size() <= 40;
      allow read, update, delete: if isAdmin();
    }
  }
}
```

## 4. Host on GitHub Pages
1. Create a public repo and upload `index.html` and this README.
2. **Settings > Pages**: Source = *Deploy from a branch*, Branch = `main` / `(root)`, Save.
3. Site is live at `https://<your-username>.github.io/<repo-name>/`.

## How it behaves
- Always shows the **next Wednesday** (today, if it's Wednesday). Thursday rolls over to the following week.
- Visitors: sign up with name + email, see who's playing. Nothing else.
- You: click **Admin sign in** at the bottom, then you see emails, a "Showed up" checkbox and Remove buttons.
- Old weeks stay in the database (view them in the Firestore console under `signups`).

## Notes
- The Firebase config in `index.html` is safe to be public. The rules are the real security.
- Emails live in a separate `contacts` collection that only you can read. You can view or export them in the Firebase console.
- Anyone could still add fake sign-ups (no login for visitors). You can remove them with your admin account.
