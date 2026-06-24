- Page 42: Replace `ng add @angular/fire` with:
    - Navigate at your *easymenu* Firebase project through the Firebase Console.
    - Click the *Add app* button in the Project Overview page.
    - Select the web app icon.
    - Enter `easymenu` as the *App nickname* and click the *Register app* button.
    - Run `npm install firebase @angular/fire -f`.
    - Open the `app.config.ts` file.
    - Paste the Firebase configuration contents.
    - Add the following statement:
        ```ts
        import { getFirestore, provideFirestore } from '@angular/fire/firestore';
        ```
    - Add the following in the `providers` array:
        ```ts
        provideFirebaseApp(() => app),
        provideFirestore(() => getFirestore())
        ```
- Page 43: Remove `projectNumber` and `version` from the Firebase configuration object