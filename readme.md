## setup your firebase project in the firebase admin UI

- create a firebase account at console.firebase.google.com
- create a project
- select the project

  - create a firebase web app within the project
  - create auth, firestore, storage and functions in the project

## setup your project

- create your new project directory and initialise it: `npm init`
- install firebase-tools

  - as a project dependency: `npm install -D firebase-tools`
  - globally: `npm install -g firebase-tools`
    - this allows for firebase commands to be run with either `firebase` or `node_modules/.bin/firebase`. For example;
      - `firebase init` or `node_modules/.bin/firebase init`
      - to make it easier add any scripts to `package.json` script as follows: `"firebase:init": "node_modules/.bin/firebase init"`
    - all firebase commands in this readme are stored in the `package.json` scripts section to make it easier to run

## setup firebase within your project

- login to firebase: `firebase login`
  - follow the cli wizard
- initialize firebase project: `firebase init`. Some steps missing, use the default values and install all dependencies

  - from the first products selector select firestore, functions, storage and emulators (auth probably won't be in the list)
  - from the emulator products selector select auth, firestore, functions, storage and emulators
  - When you get to functions, select typescript as the language
  - continue with CLI wizard until setup is complete

## functions

- navigate to the functions directory and compile typescript on save: `cd functions && npm run build:watch`
  - go to `functions/src/index.ts` and uncomment the helloWorld function
  - you may find that you get errors in the IDE, this can be resolved with the following
    - copy the `tsconfig.json` file from the functions directory to the root of the project
    - edit the new `tsconfig.json` file to prefix any file paths with `functions/`
    - if you get an error from the IDE when accessing the `.eslintrc.js` file, change the file name to `.eslintrc.mjs` and change the `module.exports = {` to `export default {`

## tidy

- in the `firebase.json` file, there are many file paths that need to be updated to improve the project structure
  - update the `firestore.rules` path to `firestore/firestore.rules` and do the same for indexes then move the files to a `firestore` directory
  - update the `storage.rules` path to `storage/storage.rules` then move the files to a `storage` directory

## history

- create a cache directory in the root of the project: `mkdir caches`
  - create a history directory in the caches directory: `mkdir caches/history`
  - create a seeds directory in the caches directory: `mkdir caches/seeds`
  - summarised by: `mkdir caches caches/history caches/seeds`
- create a firebase cache:
  - run `npm run firebase:emulator:export-on-exit`
  - stop above command with ctrl+c / cmd+c
    - this will create a cache file in the `caches/history` directory
- import the cache by moving the file to the `caches/seeds` directory and renaming it to `seed1`
- don't unnecessarily add caches to git by doing the following:
  - add a blank file `caches/history/.gitkeep`
  - add the following lines to `.gitignore` file:
    - `caches/history/*`
    - `!caches/history/.gitkeep`
- You can now run `npm run firebase:emulator:cache` as your default emulator command. It does the following:
  - on start: loads the seed file
  - on stop: saves a cache to `caches/history`

## connecting your FE to the emulator

- prefix your dev-server start command with `SOME_ENV_KEY=true` to enable the environment variable `SOME_ENV_KEY` and set it to `true`. For example:
  - if your dev-server start command is `npm run dev` then change it to `USE_FIREBASE_EMULATOR=true npm run dev`
  - you may need to add a prefix to get this to work on some frameworks. If you're using Next.js, you will need the command to be `NEXT_PUBLIC_USE_FIREBASE_EMULATOR=true npm run dev`
- in the `firebase:emulator` script in the `package.json` we have used the name demo-project. Get the firebase config from project settings and change the name for each value in the config to `demo-project`.
- initialise the app: `const app = initializeApp(firebaseConfig);`
- get auth,firestore,storage,functions: `const db = getAuth(app);` etc.
- connect auth,firestore,storage,functions to emulator: `connectAuthEmulator(auth, "http://127.0.0.1:9099");` etc.
- The resulting file should look like this:

```ts
import { initializeApp } from "firebase/app";
import { connectAuthEmulator, getAuth } from "firebase/auth";
import { connectFirestoreEmulator, getFirestore } from "firebase/firestore";
import { connectStorageEmulator, getStorage } from "firebase/storage";
import { connectFunctionsEmulator, getFunctions } from "firebase/functions";

const useEmulator = process.env.NEXT_PUBLIC_USE_FIREBASE_EMULATOR === "true";

const prodFirebaseConfig = {
  projectId: "encryptdrop",
  apiKey: "AIzaSyCewPJhmBtpn6QWXhx2MRrtY7UxaOZfOW4",
  authDomain: "encryptdrop.firebaseapp.com",
  storageBucket: "encryptdrop.firebasestorage.app",
  messagingSenderId: "352557738491",
  appId: "1:352557738491:web:6cf71ea7f33e25475a835e",
};
const emulatorProjectId = "demo-project";
const emulatorFirebaseConfig = {
  projectId: emulatorProjectId,
  apiKey: emulatorProjectId,
  authDomain: emulatorProjectId,
  storageBucket: emulatorProjectId,
  messagingSenderId: emulatorProjectId,
  appId: emulatorProjectId,
};
const firebaseConfig = useEmulator
  ? emulatorFirebaseConfig
  : prodFirebaseConfig;
export const app = initializeApp(firebaseConfig);

export const auth = getAuth(app);
export const db = getFirestore(app);
export const storage = getStorage(app);
export const functions = getFunctions(app);

if (useEmulator) {
  connectAuthEmulator(auth, "http://127.0.0.1:9099");
  connectFirestoreEmulator(db, "127.0.0.1", 8080);
  connectStorageEmulator(storage, "127.0.0.1", 9199);
  connectFunctionsEmulator(functions, "127.0.0.1", 5001);
}
```
