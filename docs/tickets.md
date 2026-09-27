# Porilo — ticket backlog

## Name

**Porilo** = *pora* (Polish: "time of day", as in *pora dnia*) + *-ilo* (Esperanto suffix meaning "tool").
Together: *a tool for the time of your day*.

The name has a Polish root without sounding Polish, and reads naturally in English (POH-ree-loh). The Esperanto suffix is a nod to Poland too: Esperanto was created by Ludwik Zamenhof from Białystok.

| Item | Value |
|---|---|
| App name (under the icon) | Porilo |
| Store title | Porilo: Offline Day Planner |
| GitHub organization | `porilo-app` (github.com/porilo-app) — `porilo` was taken |
| App repo | `porilo-app/porilo-mobile` |
| Website / privacy policy | `porilo-app/porilo-app.github.io` → porilo-app.github.io |
| Package name | `io.github.porilo_app` or `pl.<yourname>.porilo` (decided in T02; cannot be changed after the first Play upload) |
| Deep link scheme | `porilo://` |

Before committing to it, check it in Google Play search, and search TMview (EU trademark search).

## About the app

The app is an offline day planner built in React Native, Android first. It has 3 palettes, each in light and dark, 4 text sizes, and PL/EN. Reminders are local, with no backend, no account and no analytics, as the design specifies.

## Confirmed decisions

- ✅ **Expo + dev build + CNG (`expo prebuild`).** The `android/` folder is generated, never committed. Expo Go is not an option, because notifee needs native code. **No paid EAS services** — building and publishing run on GitHub Actions.
- ✅ **Android now, iOS later.** Ground rules for later: never commit `ios/`, check iOS support for every library, keep Android-specific code in `*.android.ts(x)` files, and keep iOS shadow values in the tokens.
- ✅ **Maestro** for E2E. The CLI is free, flows are YAML, and it works well with Expo and on the emulator in GitHub Actions. It supports iOS too.
- ✅ **Icons: Phosphor** (`phosphor-react-native`, MIT).
- ✅ **SQLite for all storage** (`expo-sqlite`): events in tables, settings in `expo-sqlite/kv-store`. One storage technology.
- ✅ **No network, ever.** Nothing is sent to or stored on any server — not ours, not Google's backup. The release build doesn't even have the `INTERNET` permission, and CI enforces it (T13a).
- **Trunk-based development**: short-lived branches, PRs into `main`, squash merge, PR titles in Conventional Commits format.

Other decisions are marked **[DECISION]** in the tickets below, each with a recommendation.

## Costs

- Expo (the framework), every library in this plan, Maestro CLI, Phosphor: **free** (open source).
- GitHub + GitHub Actions for a public repo: **free** (standard runners).
- Google Play Console: **one-time registration fee**.
- iOS later: Apple Developer Program annual fee. macOS runners are free for public repos.
- Deliberately skipped: EAS Build/Submit/Update, Maestro Cloud, Sentry.

## How to read the tickets

Each ticket has four parts:
- **Description:** what the ticket covers and why.
- **Suggestions:** direction, libraries and pitfalls, with no ready-made code, since you are writing it yourself.
- **Acceptance criteria:** a checklist you can paste into a GitHub Issue.
- **Dependencies and size:** S ≈ one evening, M ≈ 2–3 evenings, L ≈ a week.

The order is deliberate. Fundamentals and a **walking skeleton** come first: an empty app that goes through the full CI, E2E and lands on the Google Play internal track. Features come after that. Every feature is then tested and shipped through the same pipeline, and going to production at the end is just a track promotion.

> ⚠️ **Do T07 as early as possible, in parallel with everything else.** New personal Google Play developer accounts must run a closed test with at least 12 testers for 14 days before they get production access. Check the current requirements, because Google has changed them before. Identity verification also takes time.

---

## Phase 0 — Fundamentals

### T01 · Repository and working agreements
**Description:** Set up a public GitHub repo with the basic files and branch protection, so work goes through PRs from the very first commit.

**Suggestions:**
- **[DECISION] License:** MIT is the simplest; choose GPL-3.0 if you want forks to stay open.
- **GitHub organization `porilo-app`:** create a free organization (github.com/porilo-app; `porilo` was taken) and put every project in it as a separate repo. The app lives in **`porilo-app/porilo-mobile`**. Future projects get their own repos next to it, e.g. `porilo-app/porilo-app.github.io` for the website and privacy policy (T45).
- **Why an organization:** a brand URL independent of your personal account, a clear place for future repos, the option to add collaborators with per-repo permissions, and a free GitHub Pages domain `porilo-app.github.io`. GitHub Free for organizations covers unlimited public repos, and Actions are free for public repos.
- **Organization settings:** require 2FA for members, keep base member permissions at "Read", and enable Dependabot alerts, secret scanning and push protection as defaults for new repos. Keep release secrets in the `porilo-mobile` repo's `release` environment (T12), not as organization-wide secrets.
- **Future repos and the privacy promise:** a website, docs or an iOS app fit the "no network" guarantee. A backend service would contradict it (T13a) — if one ever appears, it needs its own ADR.
- **Sharing code between repos:** with separate repos, shared code has to be published as a package (e.g. GitHub Packages) and versioned. Don't do this upfront; only when a second repo really needs the same code.
- **README:** one sentence about the project, the name explanation (see "Name" above), a "how to run" section (filled in in T02) and a link to the design.
- **Templates:** `.github/ISSUE_TEMPLATE/ticket.md` with the sections used in this document, and `.github/pull_request_template.md` (what, why, how tested, screenshots).
- **Branch protection on `main`:** PR required, required checks (added in T05), no force pushes, squash merge only.
- **Repo settings:** enable Dependabot alerts, secret scanning and push protection.
- **Labels:** `phase-0`…`phase-7`, `bug`, `tech-debt`. Milestones map to phases.

**Acceptance criteria:**
- [ ] Public repo with a license, README and `.gitignore` (Node + Android + Expo)
- [ ] Issue and PR templates
- [ ] `main` protected; merges only via PR (squash)
- [ ] Secret scanning and push protection enabled

**Dependencies:** none · **Size:** S

---

### T02 · Initialize the Expo + TypeScript project
**Description:** An empty project that runs as a dev build on the emulator and on your phone.

**Suggestions:**
- **Location:** the app sits at the root of the `porilo-app/porilo-mobile` repo — no subfolders or workspaces needed.
- **TypeScript:** use `create-expo-app` with the TypeScript template. Turn on `strict: true` in `tsconfig.json`, and consider `noUncheckedIndexedAccess`.
- **Dev build:** install `expo-dev-client` and build locally with `npx expo run:android`. You need Android Studio, the Android SDK, and the JDK version your Expo SDK requires.
- **CNG:** add `android/` and `ios/` to `.gitignore`. All native configuration lives in `app.json`/`app.config.ts` and config plugins.
- **[DECISION] Package name:** set `android.package` now (`io.github.porilo_app` — you control porilo-app.github.io through the organization's GitHub Pages; the hyphen becomes an underscore because package names can't contain hyphens — or `pl.<yourname>.porilo`). **It cannot be changed after the first upload to Play.**
- **Node version:** pin it in `.nvmrc` and the `engines` field of `package.json`. CI will use the same version.
- **Package manager:** pick npm, pnpm or yarn and stick with it. pnpm tends to need extra configuration in RN, so npm is the safest start.

**Acceptance criteria:**
- [ ] `npx expo run:android` builds and launches the app on the emulator
- [ ] TypeScript strict; `tsc --noEmit` passes
- [ ] Package name set in the config
- [ ] README: "requirements and running" section

**Dependencies:** T01 · **Size:** S

---

### T03 · Linting, formatting, git hooks
**Description:** Automatic code consistency before there is much code.

**Suggestions:**
- **ESLint:** flat config with `eslint-config-expo` as the base, plus React Hooks rules and type-aware `@typescript-eslint`.
- **Prettier:** run it separately, not as an ESLint rule. Add `eslint-config-prettier` so the two don't conflict.
- **Pre-commit:** `husky` + `lint-staged`, run on changed files only so it stays fast.
- **Commit messages:** `commitlint` with the conventional config. Or skip it locally and only check the PR title in CI (T05), since you squash merge.
- **Scripts in `package.json`:** `lint`, `format`, `format:check`, `typecheck`.

**Acceptance criteria:**
- [ ] `npm run lint`, `format:check`, `typecheck` run and pass
- [ ] Pre-commit runs lint-staged
- [ ] No ESLint/Prettier rule conflicts

**Dependencies:** T02 · **Size:** S

---

### T04 · Unit tests: Jest + React Native Testing Library
**Description:** Test infrastructure, a first test of a pure function, and a first component test.

**Suggestions:**
- **Setup:** `jest-expo` preset and `@testing-library/react-native`. Matchers like `toBeVisible` are built into recent versions.
- **Time zone:** set it in the test script (`TZ=Europe/Warsaw`). In a calendar app, tests without a fixed time zone will randomly fail in CI.
- **Global mocks:** put them in `jest.setup.ts`, and mock native modules there as they get added.
- **Coverage:** `lcov` report with a low threshold to start (e.g. 60%), raised over time. Exclude config files and placeholder screens.
- **File convention:** tests next to the file (`foo.test.ts`) or in `__tests__` — pick one.

**Acceptance criteria:**
- [ ] `npm test` runs tests; `npm run test:coverage` produces a report
- [ ] One pure-function test and one component render test
- [ ] Tests are deterministic regardless of the machine's time zone

**Dependencies:** T02 · **Size:** S

---

### T05 · CI #1: checks on every PR
**Description:** A GitHub Actions workflow that blocks merging when something is wrong.

**Suggestions:**
- **Path filters:** use `paths-ignore` (e.g. `docs/**`, `*.md`) so documentation-only changes don't trigger the expensive E2E job from T11.
- **Workflow:** `.github/workflows/pr.yml` on `pull_request` to `main`. Jobs `lint`, `typecheck`, `format:check` and `test` (coverage as an artifact) can run in parallel.
- **Node setup:** `actions/setup-node` with `node-version-file: .nvmrc` and the built-in npm cache, then `npm ci`.
- **PR title check:** enforce Conventional Commits (e.g. `amannn/action-semantic-pull-request`). The changelog in T44 is generated from PR titles.
- **Concurrency:** use `concurrency` with `cancel-in-progress`, so a new push cancels the old run.
- **Public-repo security:** never use `pull_request_target` with a checkout of PR code. Default to `permissions: contents: read`.
- **Required checks:** after the first green run, mark the jobs as required in branch protection.

**Acceptance criteria:**
- [ ] Every PR runs lint, typecheck, format and tests
- [ ] Checks are required for merge
- [ ] An invalid PR title blocks merge

**Dependencies:** T03, T04 · **Size:** S

---

### T06 · Architecture and folder structure (ADRs)
**Description:** Write down a few decisions before building features, so you don't reinvent them in every PR.

**Suggestions:**
- **ADRs:** a `docs/adr/` folder with short records (context, decision, consequences). First ones: Expo + CNG, folder structure, date storage, state library, E2E.
- **Proposed layout:**
  - `src/domain`: pure TS with no React — types, validation, date logic.
  - `src/data`: SQLite and repositories.
  - `src/ui`: the design system.
  - `src/features/<screen>`: screens and their hooks.
  - `src/i18n` and `src/notifications`.
- **Layer rule:** `domain` never imports from `data` or `ui`. Enforce it with an ESLint rule (`import/no-restricted-paths`).
- **Path aliases:** set them in `tsconfig` (`@/domain/...`); Expo supports them natively.
- **Future reuse:** the ESLint layer rule keeps `src/domain` free of React Native. If another repo ever needs the same logic, extracting it into a published package will be mechanical.

**Acceptance criteria:**
- [ ] At least 3 ADRs in the repo
- [ ] Folder structure created; aliases work in the app and in Jest
- [ ] ESLint rule enforces layer boundaries

**Dependencies:** T02 · **Size:** S

---

### T07 · Google Play Console account *(in parallel, as early as possible)*
**Description:** Create and verify the developer account. This takes time and doesn't depend on any code.

**Suggestions:**
- **Registration:** one-time fee, identity verification (ID document), phone/device verification.
- **[DECISION] Personal vs organization account:** an organization needs a D-U-N-S number but skips the closed-testing requirement.
- **Testers:** check the current requirements for new personal accounts (closed test, number of testers, number of days). Start gathering testers now — friends with an Android phone and a Google account.
- **App entry:** create the app in the console (name "Porilo", default language Polish, free).

**Acceptance criteria:**
- [ ] Account verified
- [ ] App created in the console
- [ ] List of at least 12 people willing to join the closed test

**Dependencies:** none · **Size:** S (waiting time: days)

---

## Phase 1 — Walking skeleton

### T08 · Navigation and placeholder screens
**Description:** Every screen from the design exists as an empty view with a title, and you can navigate between them.

**Suggestions:**
- **Screens:** Day (home), Month, New/Edit event (one screen, mode set by a parameter), Details, Settings, First run.
- **[DECISION] Router:** Expo Router gives file-based routing and free deep links. React Navigation is more manual configuration but more explicit. Recommendation: Expo Router.
- **Deep links:** define the scheme now, e.g. `porilo://event/<id>`. The notification in T40 will target it.
- **Onboarding:** a conditional redirect. The "completed" flag arrives in T24; hardcode it for now.
- **`testID`s:** add them to key elements now. Convention e.g. `screen-day`, `fab-add`, `btn-save`.

**Acceptance criteria:**
- [ ] 6 screens with transitions as in the design (FAB → form, card → details, app bar → month/settings)
- [ ] Deep link `porilo://event/123` opens the details screen (verified with `adb shell am start`)
- [ ] `testID` convention documented in the README or an ADR

**Dependencies:** T06 · **Size:** M

---

### T09 · Internationalization PL/EN
**Description:** All strings come from translation files. The language follows the system or is picked manually.

**Suggestions:**
- **Libraries:** `i18next` + `react-i18next`, plus `expo-localization` to detect the system language.
- **Polish plurals:** keys `_one`, `_few`, `_many` (1 wydarzenie / 3 wydarzenia / 5 wydarzeń). i18next relies on `Intl.PluralRules`; verify that Hermes supports it in your version.
- **Dates and weekday names:** `date-fns` with the `pl`/`enUS` locales, or `Intl.DateTimeFormat`. The first day of the week also depends on the locale, as the design points out.
- **Typed keys:** i18next has a type declaration mechanism for this, so a typo in a key is a compile error.
- **Key parity test:** a unit test that both language files have exactly the same set of keys.

**Acceptance criteria:**
- [ ] No hardcoded strings in components
- [ ] Changing the system language changes the app language ("Follow system" mode)
- [ ] PL/EN key parity test
- [ ] Plural tests for 1, 2, 5, 12, 22

**Dependencies:** T08 · **Size:** M

---

### T10 · Local E2E: Maestro
**Description:** First E2E flow on the placeholders: launch the app, go to settings, come back.

**Suggestions:**
- **Setup:** Maestro CLI, flows in `.maestro/`.
- **Selectors:** use `id:` (i.e. `testID`), not text, because text changes with the language.
- **Build type:** run E2E against a **release** build (or a separate variant without the dev menu), not a dev build with Metro — the same as CI.
- **Isolation:** every flow starts with `clearState`, so it doesn't depend on earlier ones.
- **Script:** `npm run e2e`.

**Acceptance criteria:**
- [ ] `npm run e2e` passes locally on the emulator
- [ ] Flows use `id` selectors only
- [ ] How to run it is documented in the README

**Dependencies:** T08 · **Size:** S

---

### T11 · CI #2: E2E on an emulator
**Description:** Build an APK and run the Maestro tests in GitHub Actions on every PR.

**Suggestions:**
- **Build job:** on `ubuntu-latest`, enable KVM (a udev rule step, documented in the emulator action's README), then `actions/setup-java`, `npx expo prebuild --platform android`, and `./gradlew assembleRelease`.
- **Signing:** sign the E2E release build with the debug key. The real key is only used in the release workflow, and secrets aren't available to PRs from forks anyway.
- **Emulator:** `reactivecircus/android-emulator-runner`. Cache the AVD snapshot and Gradle (`gradle/actions/setup-gradle`); without caching the job will be very slow.
- **Artifacts on failure:** Maestro recordings/screenshots and logcat.
- **[DECISION] When to run:** if job time becomes painful, run E2E only on labelled PRs, or nightly on `main`. Start with every PR.

**Acceptance criteria:**
- [ ] E2E job passes in CI and is a required check
- [ ] Diagnostic artifacts on failure
- [ ] Gradle and AVD caching works (the second run is noticeably faster)

**Dependencies:** T05, T10 · **Size:** M

---

### T12 · Signing and AAB build in CI
**Description:** A workflow that builds a signed Android App Bundle ready for Play.

**Suggestions:**
- **Upload key:** generate it with `keytool`. Store the keystore and passwords in a password manager, plus an offline backup. If you lose it, you go through Google's reset procedure.
- **Play App Signing:** enable it (the default for new apps). Google holds the app signing key; you only keep the upload key.
- **GitHub secrets:** keystore as base64, passwords, alias. Ideally in a **GitHub Environment** named `release` with you as a required reviewer.
- **Injecting signing config:** since `android/` is generated, inject it with **your own small config plugin** (good practice), or via `gradle.properties` from env vars in CI. Never commit the keystore.
- **Versioning:** `versionCode` must increase monotonically, e.g. from `github.run_number` or a counter in the repo. `versionName` comes from `package.json`.

**Acceptance criteria:**
- [ ] `release.yml` workflow (manual `workflow_dispatch` for now) builds a signed `.aab` as an artifact
- [ ] No keys in the repo or in logs
- [ ] `versionCode` increments automatically

**Dependencies:** T11 · **Size:** M

---

### T13 · Automatic upload to the internal track
**Description:** The app skeleton lands on Google Play (Internal testing) from the pipeline.

**Suggestions:**
- **First upload:** **upload the first AAB manually** in the console. The API won't accept uploads for an app that has no build yet.
- **Service account:** create one in Google Cloud and enable the Google Play Android Developer API. Add the account's email under Play Console → Users and permissions with release permissions. Store the JSON key as a secret in the `release` environment.
- **Upload step:** the `r0adkll/upload-google-play` action or fastlane `supply`. Internal track only at this stage.
- **Try it yourself:** add yourself as an internal tester and install the app from Play on your phone.

**Acceptance criteria:**
- [ ] Workflow uploads the AAB to the internal track
- [ ] App installs from Play on your phone
- [ ] Secrets live only in an environment with required approval

**Dependencies:** T07, T12 · **Size:** M

---

### T13a · Privacy by design: no network, no cloud backup
**Description:** Make "nothing ever leaves the device" a guarantee enforced by the OS and by CI, not just a promise in the README.

**Suggestions:**
- **Remove the `INTERNET` permission in release builds.** React Native adds it by default. Use `android.blockedPermissions` in `app.config.ts` (also `ACCESS_NETWORK_STATE`). Make it conditional on a build variant env var (e.g. `APP_VARIANT`), because the dev build needs network to talk to Metro. Without this permission the OS itself blocks any network call, including from a dependency you didn't audit.
- **CI permission guard:** after building the release APK, dump its permissions (`aapt2 dump permissions` or `apkanalyzer manifest permissions`) and compare against an allowlist in the repo, e.g. `POST_NOTIFICATIONS`, the exact-alarm permission, `RECEIVE_BOOT_COMPLETED`, `VIBRATE`. Any new permission fails the build until it's reviewed and added to the allowlist on purpose.
- **Disable Android Auto Backup:** `android.allowBackup: false`. Otherwise Android backs up the app's data, including the SQLite file, to the user's Google Drive by default. Read the current Android docs on device-to-device transfer: on newer Android versions `allowBackup=false` doesn't cover it, and you control it through data extraction rules. D2D is a local transfer, not a server, so decide deliberately.
- **No phoning-home libraries:** never install `expo-updates`, analytics or crash-reporting SDKs. Add a PR template checkbox: "Does this add a dependency with network access or a new permission?"
- **[DECISION] Encryption at rest:**
  - *Default (recommended):* rely on Android's file-based encryption plus the app sandbox, which already protect the data on a locked, non-rooted phone.
  - *SQLCipher:* `expo-sqlite` supports it, with the key kept in the Android Keystore via `expo-secure-store`. It mainly adds protection on rooted devices, at the cost of complexity.
- **[DECISION] Lock-screen notifications:** show the event title on the lock screen (as in the design), or use private visibility so only "Porilo · 14:45" is visible until unlocked.
- **No user data in logs:** ESLint `no-console` plus stripping `console.*` from release builds, so event titles never end up in logcat.
- **Documentation:** an ADR "No network", a `SECURITY.md` explaining how to report vulnerabilities, and a README section explaining the privacy guarantees and how anyone can verify them in the APK.

**Acceptance criteria:**
- [ ] Release APK has no `INTERNET` permission (verified in CI)
- [ ] CI fails on any permission outside the allowlist
- [ ] `allowBackup` disabled; D2D behavior decided and documented
- [ ] ADR "No network" and `SECURITY.md` in the repo
- [ ] App works fully in airplane mode (manual check)

**Dependencies:** T11, T12 · **Size:** M

---

## Phase 2 — Design system

### T14 · Tokens and `makeTheme`
**Description:** Port section 10 of the design to code. `makeTheme(paletteId, mode, fontStep)` returns exactly that object.

**Suggestions:**
- **Contents:** pure TS in `src/ui/theme/`:
  - palettes (Ciepło, Spokój, Zieleń) × light/dark;
  - label colors (6, shared across palettes);
  - type scale (5 roles × 4 steps);
  - spacing, radius, elevation, sizing.
- **Literal types:** `PaletteId`, `Mode`, `FontStep`, `LabelId`. Adding a 7th label without light/dark values should fail to compile, as the design explicitly requires.
- **Great unit test:** implement the WCAG contrast formula. Check the pairs from the "Measured contrast" table: ≥4.5:1 for text, ≥3:1 for labels on `surface` in dark. Color regressions get caught automatically.
- **Elevation:** Android uses `elevation` and iOS uses shadows. Android-only means `elevation` is enough, but keep the iOS values in the tokens for later.

**Acceptance criteria:**
- [ ] All design tokens in code, typed
- [ ] Contrast test for all 6 themes
- [ ] Snapshot/test of `makeTheme` for every combination

**Dependencies:** T06 · **Size:** M

---

### T15 · ThemeProvider and system mode
**Description:** A theme context available across the app that follows the system mode.

**Suggestions:**
- **System mode:** use `useColorScheme()`. Memoize the theme object so a change doesn't needlessly re-render the tree.
- **API:** a `useTheme()` hook plus a small helper for theme-dependent styles, e.g. a memoized `(theme) => StyleSheet.create(...)`.
- **System bars:** status and navigation bars in theme colors. Use edge-to-edge with `react-native-safe-area-context` for insets; newer Android versions enforce edge-to-edge.
- **State:** keep the theme in memory for now; persistence comes in T24.

**Acceptance criteria:**
- [ ] Switching the system mode updates the app live
- [ ] System bars readable in both modes
- [ ] Component test with the provider in both modes

**Dependencies:** T14 · **Size:** S

---

### T16 · `Text` component and text scale
**Description:** A single text component with roles (title, heading, body, label, caption) and the user's size step.

**Suggestions:**
- **Roles:** the role picks `fontSize/lineHeight/letterSpacing/fontWeight` from the tokens for the current step.
- **[DECISION] Interaction with Android's system font size:** either multiply (app step × system scale) or cap it with `maxFontSizeMultiplier`. The design says nothing has a fixed height, so multiplying is safe. Test ×1.5 in the app plus the largest system size.
- **Letter spacing:** the design gives it in `em`-style values; convert to dp.
- **Font:** system Roboto. Don't set `fontFamily`, only `fontWeight` 400/500.

**Acceptance criteria:**
- [ ] 5 roles × 4 steps match the table
- [ ] Tests: role + step → correct styles
- [ ] Nothing clipped at ×1.5 (checked manually on a test screen)

**Dependencies:** T15 · **Size:** S

---

### T17 · UI primitives
**Description:** Button (primary/secondary/text/destructive), IconButton, FAB, Chip, Card/Surface, ListRow, SwitchRow, SegmentedControl, Divider — per section 11 of the design.

**Suggestions:**
- **Interaction:** `Pressable` with `android_ripple`. Disabled is always 45% opacity on the whole node.
- **Sizing:** `minHeight` instead of `height` everywhere, except the design's exceptions (icons, FAB, label bar, dot track).
- **Accessibility from the start:** `accessibilityRole`, `accessibilityState` (selected/disabled/checked), and `hitSlop` wherever the visual size is under 48 dp.
- **[DECISION] Showcase screen:** instead of Storybook, a hidden dev-only "Showcase" screen with every component in every state. Cheaper, and you see them on a real device.
- **Tests:** RNTL checks that `onPress` fires, doesn't fire when disabled, and `accessibilityState` is correct.

**Acceptance criteria:**
- [ ] All primitives from section 11 with states matching the design
- [ ] Showcase screen in dev
- [ ] Interaction and accessibility tests

**Dependencies:** T16 · **Size:** L

---

### T18 · Text field (filled)
**Description:** A field with the label inside, no floating labels, no outline animation.

**Suggestions:**
- **States:** focused means label in `primary` plus a 2 dp bottom border. Disabled is 45%.
- **Picker field:** a "button that looks like a field" variant (Date, From, To). Same look; it opens a picker instead of the keyboard.
- **Accessibility and errors:** `accessibilityLabel` = the field label. The validation error message sits below the field.

**Acceptance criteria:**
- [ ] TextField and PickerField match 03.1
- [ ] Tests: focus, text change, error

**Dependencies:** T17 · **Size:** S

---

### T19 · Overlays: Dialog, BottomSheet, Snackbar
**Description:** Infrastructure for section 06: scrim, a dialog that requires an answer, a sheet rounded on top, a snackbar above the FAB.

**Suggestions:**
- **[DECISION] Implementation:** your own components on `Modal` + `Animated` give fewer dependencies and full control. The design also says nothing closes by accident. `@gorhom/bottom-sheet` gives gestures for free but needs reanimated and gesture-handler. Recommendation: your own.
- **Android back button in a dialog:** decide deliberately. The design says a dialog needs an explicit answer, so back = Cancel.
- **Snackbar:** a global service (context with `show({ text, action })`), 4 s duration, `inverseSurface` colors.

**Acceptance criteria:**
- [ ] Dialog, BottomSheet, Snackbar match 06.x
- [ ] Tapping the scrim does not close a dialog
- [ ] Tests: actions fire callbacks; snackbar disappears after its timeout (fake timers)

**Dependencies:** T17 · **Size:** M

---

### T20 · Icons
**Description:** UI glyphs (app bar, detail rows) and the 12 selectable event icons.

**Suggestions:**
- ✅ **Library:** `phosphor-react-native` (MIT) plus the required `react-native-svg`.
- **Weight:** pick **one weight** (e.g. `regular`, with `fill` for selected states) and keep it in the tokens, not in components.
- **Mapping:** build a "design glyph → Phosphor icon" map for every place (app bar, detail rows, the 12 event icons). Compare on the Showcase screen, because Phosphor looks slightly different from the Material-style icons in the design.
- **Imports:** import icons individually, not the whole package, to keep the bundle small.
- **Storage:** store event icons in the database as a stable identifier (`"yoga"`, `"cake"`), not the library's name. Switching libraries won't break data.
- **Size:** fixed sizes from the tokens; icons don't scale with the text step.

**Acceptance criteria:**
- [ ] `Icon` component with typed names
- [ ] Map of the 12 event icons (id → glyph)

**Dependencies:** T14 · **Size:** S

---

### T20a · Logo and brand assets
**Description:** Design the logo and export every icon and graphic the app, Google Play and GitHub need. Design it once as a vector, then export all variants from that source.

**Assets needed:**

| Asset | Where it's used | Format & size | Notes |
|---|---|---|---|
| Master logo | source of everything else | SVG | Keep in `assets/brand/` |
| Adaptive icon — foreground | Android launcher | PNG 1024×1024, transparent | Content inside the central safe zone (~66 of 108 dp); launchers crop to circle/squircle |
| Adaptive icon — background | Android launcher | solid color or PNG 1024×1024 | E.g. Ciepło `background` or `primary` |
| Monochrome icon | Android 13+ themed icons | PNG 1024×1024, single color on transparent | The system tints it; silhouette only |
| Default icon | Expo `icon`, iOS later | PNG 1024×1024, **no transparency** | iOS rejects alpha |
| Notification small icon | status bar, notification (07.2) | PNG 96×96, white on transparent | Only the alpha channel is used; must read at 24 dp |
| Splash icon | app start (light + dark) | PNG, transparent | Android 12+ shows it inside a circle, so keep it within the inner ~2/3 |
| Play Store icon | store listing | PNG 512×512, 32-bit | Max 1 MB |
| Feature graphic | store listing | 1024×500, JPG/PNG, no alpha | Logo + name + one line of text |
| GitHub social preview | link previews of the repo | PNG 1280×640 | Repo Settings → Social preview |
| README logo | top of the README | SVG, light and dark | Use `<picture>` with `prefers-color-scheme` |

**Suggestions:**
- **Readability:** the mark must work at 48 px on a launcher and as a flat 24 dp silhouette. Test it small first, not last.
- **Colors:** derive them from the design tokens, e.g. Ciepło `primary` #9C4A06 on `background` #FDF8F4, so the icon matches the app.
- **No text in the icon**, and avoid the "calendar page with 31" cliché, which also makes the app look like Google Calendar. Play policy forbids icons that could be mistaken for another app.
- **Free tools:** Figma (free plan) or Inkscape. Preview the adaptive icon in several mask shapes before exporting.
- **Wiring:** Expo config keys `icon`, `android.adaptiveIcon` (`foregroundImage`, `backgroundColor`, `monochromeImage`), the `expo-splash-screen` plugin (with a dark variant), and the notification icon via the notification library's config plugin.
- **License:** note in the README whether the logo is covered by the repo license or kept separately. Code under MIT normally lets anyone reuse the logo too, which some projects avoid for brand assets.

**Acceptance criteria:**
- [ ] SVG source and all exports from the table in `assets/brand/`
- [ ] Launcher icon looks right with circle and squircle masks, and with themed icons on
- [ ] Splash looks right in light and dark
- [ ] Notification icon readable in the status bar
- [ ] README logo and GitHub social preview set

**Dependencies:** T14 (colors), app name chosen · **Size:** M

---

## Phase 3 — Domain and data

### T21 · Domain model and validation
**Description:** Event types and business rules, with no React and no database.

**Suggestions:**
- **`Event` fields:**
  - id, title, allDay, start, end, note?;
  - reminder (enum: none, atStart, 5m, 15m, 30m, 1h, 1d);
  - label (LabelId), icon?, createdAt, updatedAt.
- **[DECISION] Time storage** — the most important data decision. Recommendation: timed events as epoch ms (UTC), all-day events as a local `YYYY-MM-DD` date with no time. All-day as UTC midnight is the classic source of "the birthday shows up a day early" bugs.
- **Validation:** `zod` (schema = type) or hand-written functions. Rules: non-empty title, end > start, max note length.
- **TDD:** go full TDD here; it's the easiest place to get good coverage.

**Acceptance criteria:**
- [ ] Types + validation in `src/domain`
- [ ] Tests for every validation rule
- [ ] ADR on date storage

**Dependencies:** T06 · **Size:** S

---

### T22 · Date logic
**Description:** The pure functions the Day and Month screens are built on.

**Suggestions:**
- **Functions to cover:**
  - the week containing a date (first day from the locale);
  - a 5–6 row month grid including out-of-month days;
  - events on a given day, including multi-day and all-day ones;
  - sorting;
  - duration formatting ("1 godz. 30 min") and "in 15 minutes";
  - is-today.
- **Dot rule from 02:** up to 3 events = 3 dots; 4 or more = 2 dots + "+N". A pure function, easy to test.
- **DST tests:** in Poland the clocks change on the last Sunday of March and October. A day then has 23/25 hours, and a 02:30 event in March doesn't exist. Real bugs live here.
- **Inject "now":** pass it as an argument instead of calling `Date.now()` inside, so tests need no fake timers.

**Acceptance criteria:**
- [ ] All functions tested, including DST and year-boundary cases
- [ ] Month grid correct for PL (Monday) and EN (Sunday)

**Dependencies:** T21, T09 · **Size:** M

---

### T23 · Event persistence: SQLite
**Description:** An event repository on a local database, with migrations.

**Suggestions:**
- ✅ **Library:** `expo-sqlite` (confirmed).
- **[DECISION] Query layer:** either plain SQL and migrations on `PRAGMA user_version` (more learning, no magic), or Drizzle ORM (types from the schema, generated migrations). With one table, plain SQL is plenty.
- **Repository:** an `EventsRepository` interface in `domain` (getRange, getById, create, update, delete). The SQLite implementation lives in `data`.
- **In-memory implementation for tests:** `expo-sqlite` doesn't run in Node/Jest. Test the logic against the in-memory implementation and let E2E cover the real database.
- **Querying:** index the start column; the month view's range query is a single SELECT.
- **Contract test:** the same test suite runs against every repository implementation (in-memory in Jest).

**Acceptance criteria:**
- [ ] CRUD and range query work on a device
- [ ] Versioned migrations; the first one creates the schema
- [ ] Contract tests on the in-memory implementation

**Dependencies:** T21 · **Size:** M

---

### T24 · Settings persistence
**Description:** Store theme mode, palette, text step, language, onboarding completed, reminders enabled, and default reminder.

**Suggestions:**
- ✅ **Storage:** `expo-sqlite/kv-store` (AsyncStorage-like API on top of SQLite), so the app has a single storage technology.
- **No theme flash:** load settings before the first render, keeping the splash until ready.
- **Defensive reads:** a settings schema with defaults, validated on read, so old or corrupt data doesn't crash the app.

**Acceptance criteria:**
- [ ] Settings survive an app restart
- [ ] No theme flash on startup
- [ ] Tests reading missing/invalid data

**Dependencies:** T15, T09 · **Size:** S

---

### T25 · App state and data hooks
**Description:** How screens fetch and refresh data.

**Suggestions:**
- **[DECISION] State library:** `zustand` (small, simple, easy to test) or plain React Context + `useSyncExternalStore`. Either is enough here; zustand saves boilerplate.
- **Hooks:** `useEventsForDay(date)`, `useEventsForMonth(month)`, `useEvent(id)`, `useEventMutations()`. A mutation invalidates/refreshes the affected days.
- **Settings:** a separate store wired to T24.

**Acceptance criteria:**
- [ ] Saving an event refreshes the day and month views without a restart
- [ ] Hook tests with the in-memory repository

**Dependencies:** T23, T24 · **Size:** M

---

## Phase 4 — Screens

Every screen in this phase must match the design in light and dark, work at all 4 text steps, have `testID`s, and have at least one E2E flow. Add this as a criterion to each ticket.

### T26 · App bar and week strip
**Description:** The top of the day view (01).

**Suggestions:**
- **States:** a 7-cell strip where today has the `todayHighlight` tint, the selected day has a solid fill, and both can apply at once.
- **[DECISION] Switching weeks:** swipe (paginated FlatList) or arrows. The design shows a single row, so consider horizontal swipe.
- **Scaling:** the row grows downward at large steps (flexWrap / minHeight).

**Acceptance criteria:** today ≠ selected is visually distinct; tap changes the day; E2E for selecting a day · **Dependencies:** T17, T22, T25 · **Size:** M

---

### T27 · Day view (home)
**Description:** The agenda, with an all-day section, a 60 dp time gutter, cards with a label bar, the "now" line, an empty state, and the FAB.

**Suggestions:**
- **List:** use `FlatList`, not `ScrollView.map`, to handle days with many events.
- **"Now" line:** refresh it every minute. Align the interval to the full minute, clear it on unmount, and pause it in the background via `AppState`.
- **Empty state:** no illustration (an 88 dp tile + two lines).

**Acceptance criteria:** matches 01.1–01.3 and 08; "now" line test with fake timers; "empty day" E2E · **Dependencies:** T26 · **Size:** M

---

### T28 · Month view
**Description:** A grid built from nested flex rows, label dots, and a list under the grid (02).

**Suggestions:**
- **Reusable cell:** write the grid cell so the date picker (T31) can reuse it — same day model, different size.
- **Selection:** tapping a day fills the list below instead of navigating. Include a "Today" button.
- **Performance:** memoize cells (`React.memo`), since selection changes touch 42 cells.

**Acceptance criteria:** "+N" dot rule; out-of-month days in `textSecondary`; E2E selecting a day with events · **Dependencies:** T22, T25 · **Size:** M

---

### T29 · Form: new event
**Description:** Title, all-day, date, from/to, note, reminder (chips), label (scrollable chips), icon, and a bottom bar with Cancel/Save (03.1).

**Suggestions:**
- **[DECISION] Form state:** `react-hook-form` + zod resolver (consistent with T21), or `useReducer`.
- **Keyboard:** the bottom bar stays pinned above the keyboard. `react-native-keyboard-controller` handles edge-to-edge better than `KeyboardAvoidingView`.
- **All-day toggle:** "All day" disables the time fields (45%) instead of removing them.
- **Cancel with unsaved changes:** the design doesn't say whether to confirm. Decide and document it.

**Acceptance criteria:** T21 validation shown in the UI; save creates the event and returns to the day; E2E "add event → it shows up in the day" · **Dependencies:** T18, T25 · **Size:** L

---

### T30 · Time picker
**Description:** A sheet with two snapping FlatList columns, minutes in steps of 5, and fields for keyboard entry (06.2).

**Suggestions:**
- **Scroll mechanics:** `snapToInterval` = row height, `getItemLayout` for performance, and `onMomentumScrollEnd` to read the value.
- **Wrapping hours:** 23 → 0 is optional and complicated; skip it for now.
- **E2E:** column scrolling is flaky in Maestro, so drive the E2E path through the keyboard fields.

**Acceptance criteria:** picking and typing a time; 44 dp selection band; E2E setting a time · **Dependencies:** T19, T29 · **Size:** M

---

### T31 · Date picker
**Description:** A dialog with a mini month (07.1) that reuses the logic from T28.

**Acceptance criteria:** today as a ring, selection as a fill; month navigation; E2E · **Dependencies:** T19, T28 · **Size:** S

---

### T32 · Icon picker
**Description:** A sheet with 12 icons + "None". Tapping selects and closes (06.4).

**Acceptance criteria:** flexWrap grid, 5 per row at 360 dp; selected state · **Dependencies:** T19, T20 · **Size:** S

---

### T33 · Edit and delete
**Description:** The same form in edit mode, plus delete from the app bar via a confirmation dialog (03.2, 06.1).

**Acceptance criteria:** editing saves changes; delete always goes through the dialog; E2E for edit and delete · **Dependencies:** T29 · **Size:** S

---

### T34 · Event details
**Description:** A read-only view (03.3).

**Suggestions:**
- **Label chip:** dot + word, never color alone.
- **Missing events:** handle deep links to an id that no longer exists (an event deleted after its notification fired).

**Acceptance criteria:** matches 03.3; missing id → message, not a crash · **Dependencies:** T25 · **Size:** S

---

### T35 · Snackbar and undo
**Description:** "Wydarzenie zapisane · COFNIJ" ("Event saved · UNDO") (06.3).

**Suggestions:**
- **What UNDO reverts:** after create, it deletes the event; after edit, it restores the previous version.
- **Undo after delete:** often better than a dialog, but the design chose a dialog.

**Acceptance criteria:** undo restores the previous state; unit test of the undo logic · **Dependencies:** T19, T33 · **Size:** S

---

### T36 · Settings
**Description:** The settings screen (04, 05, 06.5) has four parts:
- **Appearance:** theme mode, palette cards shown in the current mode, and text size with a live preview.
- **Language:** a bottom sheet.
- **Notifications.**
- **About.**

**Suggestions:**
- **Size preview:** it renders at the selected step while the rest of the screen stays at the current one, so `Text` must accept a step override.
- **Version:** read it from `expo-application`/config; don't type it by hand.

**Acceptance criteria:** all changes apply immediately and persist; E2E "switch language → restart → still EN" · **Dependencies:** T24, T19 · **Size:** M

---

### T37 · First run
**Description:** One screen, palette choice, "Zaczynajmy" ("Let's start") (05.3). No permission prompts.

**Acceptance criteria:** shown only once; chosen palette saved; E2E with `clearState` · **Dependencies:** T24 · **Size:** S

---

## Phase 5 — Reminders

### T38 · Notification setup
**Description:** Notification library, the `reminders` channel, and a permission request the first time a reminder is set.

**Suggestions:**
- **[DECISION] Library:** notifee (as in the design; AlarmManager triggers, actions, background handling) or `expo-notifications`. Before choosing, check repo activity and compatibility with your RN version and the New Architecture.
- **Permission:** Android 13+ needs the runtime `POST_NOTIFICATIONS` permission. Handle denial with a note in settings and a link to the system settings.
- **Exact alarms:** read the current Play policy on `SCHEDULE_EXACT_ALARM` / `USE_EXACT_ALARM`. Calendar apps have special status here, but it requires a declaration in the console. Without exact alarms, reminders can be late in Doze mode.
- **Small notification icon:** from T20a, added via the library's config plugin.

**Acceptance criteria:** channel created; permission asked only on the first reminder; denial handled · **Dependencies:** T29 · **Size:** M

---

### T39 · Reminder scheduling
**Description:** Creating, editing or deleting an event keeps the scheduled notifications in sync.

**Suggestions:**
- **One sync function:** a single `reconcile(event)` instead of scattered calls — cancel the old, schedule the new (notification id = event id).
- **Full sync on startup:** e.g. after a backup restore or an update. Check the behavior after a phone reboot.
- **Edge cases:** past reminders are not scheduled. The global switch in settings cancels everything.
- **Testability:** extract "what to schedule and when" as a pure function and unit test it, with library calls mocked.

**Acceptance criteria:** reconcile tests (create, time change, change to "None", delete, past); manual test with a reminder 1 min ahead · **Dependencies:** T38 · **Size:** M

---

### T40 · Notification actions and deep link
**Description:** Content as in 07.2, actions "Snooze 10 min" and "Open", and a tap that opens the event details.

**Suggestions:**
- **Snooze in the background:** "Snooze" must work while the app is killed, so register the background handler outside the React tree.
- **Open:** `porilo://event/<id>` from T08.

**Acceptance criteria:** snooze works with the app closed; tap opens the right event · **Dependencies:** T39, T34 · **Size:** M

---

## Phase 6 — Quality

### T41 · Full E2E suite
**Description:** Fill in the flows for the critical scenarios.

**Suggestions:**
- **Flows to cover:**
  - CRUD;
  - form validation;
  - day ↔ month navigation;
  - settings persistence across restart (`stopApp`/`launchApp`);
  - EN language;
  - ×1.5 step (is "Save" still reachable?).
- **Notifications:** skip them in E2E, as they're too flaky. Keep a manual test in the release checklist instead.

**Acceptance criteria:** at least 8 flows; stable over 10 consecutive CI runs · **Dependencies:** phase 4 · **Size:** M

---

### T42 · Accessibility and scaling
**Description:** Walk through the whole app with TalkBack and test at the largest sizes.

**Suggestions:**
- **TalkBack:** focus order, labels on icon buttons, snackbar announcements (`accessibilityLiveRegion`), and grouping each event card into one element.
- **Sizes:** 48 dp touch targets; check ×1.5 together with the largest system font.

**Acceptance criteria:** TalkBack checklist in `docs/`; no clipped text on the screens from section 08 · **Dependencies:** phase 4 · **Size:** M

---

### T43 · Repo maintenance
**Description:** Automated dependency updates and security scanning.

**Suggestions:**
- **Dependabot:** group Expo updates. Update Expo packages together via `npx expo install --fix`, not one by one.
- **Other:** CodeQL for JS/TS, a CI badge in the README, `CONTRIBUTING.md`.

**Acceptance criteria:** Dependabot opens PRs; CodeQL runs in CI · **Dependencies:** T05 · **Size:** S

---

### T43a · (optional) Automated AI PR review
**Description:** Since AI will do the reviews, you can hook it into PRs instead of pasting diffs by hand.

**Suggestions:**
- **Setup:** a GitHub Action triggered on PRs, with the API key as a secret. In a public repo, PRs from forks won't get the secret, and that's a good thing.
- **Free alternative:** paste `git diff main...` into a chat.

**Dependencies:** T05 · **Size:** S

---

## Phase 7 — Production release

### T44 · Versioning and changelog
**Description:** Version and release notes generated from PR history.

**Suggestions:**
- **release-please:** it maintains a "release" PR, bumps `package.json`, and creates a tag and a GitHub Release with a changelog from Conventional Commits titles.
- **Trigger:** a `v*` tag triggers `release.yml` from T12/T13 instead of a manual dispatch.

**Acceptance criteria:** merging the release PR → tag → AAB on internal automatically · **Dependencies:** T13 · **Size:** S

---

### T45 · Store listing and Play requirements
**Description:** Everything the console requires before leaving internal testing.

**Suggestions:**
- **Graphics:** everything comes from T20a (512 px icon, 1024×500 feature graphic).
- **Screenshots:** generate them with Maestro (`takeScreenshot`) on seeded data, so they're reproducible after every UI change.
- **Texts:** PL and EN descriptions, and a privacy policy hosted on porilo-app.github.io (repo `porilo-app/porilo-app.github.io`) ("data stays on the device, nothing is collected").
- **Console forms:**
  - Data safety (nothing collected);
  - content rating and target audience;
  - ads: none;
  - exact alarm declaration (T38).
- **Target SDK:** a `targetSdkVersion` that meets Play's current requirement. Check on release day.

**Acceptance criteria:** every "App content" section in the console is green · **Dependencies:** phase 4, T20a, T38, T13a · **Size:** M

---

### T46 · Closed testing
**Description:** A closed testing track with the required number of testers for the required period.

**Suggestions:**
- **Testers:** manage the list via a Google Group or emails. Testers must opt in and keep the app installed.
- **Feedback:** collect it in GitHub Issues with a "tester bug report" template.
- **Promotion:** internal → closed via a workflow with a track parameter.

**Acceptance criteria:** required period completed; production access application submitted · **Dependencies:** T45 · **Size:** S (+ waiting time)

---

### T47 · Production 🚀
**Description:** The first production release and the process for the ones after it.

**Suggestions:**
- **Promotion workflow:** `promote.yml` (`workflow_dispatch`, environment with approval) moves the approved build from closed/internal to production. It ships the same binary that passed testing instead of a new build.
- **Staged rollout:** e.g. 20% → 100%, as a workflow parameter.
- **Release checklist** in `docs/release.md`:
  - manual reminder test;
  - phone reboot;
  - upgrade from the previous version (database migrations!).
- **Monitoring:** Android Vitals in the console (crashes, ANRs). No Sentry or analytics, keeping the "no analytics" promise.

**Acceptance criteria:**
- [ ] App publicly available on Google Play
- [ ] One-click promotion to production with approval
- [ ] Release checklist in the repo

**Dependencies:** T46 · **Size:** M
