# Routing & Automation Options: Zoho WorkDrive / Google Drive / iCloud / QNAP NAS / Apple Silicon Macs (state as of 2026-10-08)

Research method note: ~45 web searches were run; direct page fetches (rclone.org, help.zoho.com, noodlesoft.com, qnap.com, support.google.com, sixcolors.com) were blocked by the sandbox proxy (DNS/403), so every finding below rests on search-result excerpts of those pages rather than full page reads. Where only a snippet was visible, that is flagged. The report writer should treat version numbers as "latest seen in a source," not necessarily the latest shipped.

---

## Key Question 1: Zoho WorkDrive — TrueSync for macOS, API/WebDAV, rclone "zoho" backend, Zoho Flow, and QNAP HBS 3 reachability

### Takeaway
Zoho WorkDrive is the hardest node in this topology to automate from outside: the Mac client (TrueSync) is a File Provider "virtual drive" with online-only files by default; there is no WebDAV endpoint (only years-old, unanswered feature requests); rclone's `zoho` backend exists but is crippled by Zoho API throttling and lacks modtime/checksum support; and QNAP HBS 3 has no Zoho provider, so the NAS cannot talk to WorkDrive directly — the only practical NAS↔WorkDrive bridge is a Mac running TrueSync (or a throttled rclone job).

### Cited Findings

**TrueSync desktop app (macOS)**
- Zoho's current help article says the macOS app "has been redesigned based on the Apple File Provider Extension (FPE)" so users can "access, edit, manage, and synchronize WorkDrive files and folders without needing to enable any third-party extensions"; supported OS listed as "macOS 12 (Monterey OS) and above." — [Zoho: Install WorkDrive Desktop App (TrueSync) for macOS](https://help.zoho.com/portal/en/kb/workdrive/desktop-apps/workdrive-truesync/articles/truesync-for-mac)
- Selective sync model: "By default, all your files and folders will only be available online only"; you right-click items in the TrueSync location in Finder and choose "Make offline" / "Make online" (multi-select allowed); "When you upload files into an offline folder, those files will be made offline." — [Zoho TrueSync for Mac](https://help.zoho.com/portal/en/kb/workdrive/desktop-apps/workdrive-truesync/articles/truesync-for-mac)
- Limitation stated on the same page: "Currently, sync folder location and cache storage location cannot be changed in macOS." — [Zoho TrueSync for Mac](https://help.zoho.com/portal/en/kb/workdrive/desktop-apps/workdrive-truesync/articles/truesync-for-mac)
- The older "WorkDrive Sync" app (pre-TrueSync, mirror-style) still exists and can be installed alongside/instead of TrueSync: "If you already have the older WorkDrive Sync app, you can still install and use the WorkDrive TrueSync app... use either app and uninstall the other." — [Zoho TrueSync for Mac](https://help.zoho.com/portal/en/kb/workdrive/desktop-apps/workdrive-truesync/articles/truesync-for-mac); [Install and set up WorkDrive Sync for Mac](https://help.zoho.com/portal/en/kb/workdrive/desktop-apps/sync/workdrive-sync/articles/install-and-set-up-workdrive-sync-for-mac)
- Zoho published a dedicated "WorkDrive TrueSync for macOS 26 (Tahoe) Beta" thread: "a dedicated beta build of WorkDrive TrueSync, tailored specifically for macOS 26 compatibility," "intended for users running macOS 26 beta only," warning that "occasional glitches or limitations may occur." The thread's user replies were not retrievable. — [Zoho Community: TrueSync for macOS 26 (Tahoe) Beta](https://help.zoho.com/portal/en/community/topic/workdrive-truesync-for-macos-26-tahoe-beta)
- Reliability complaints (Zoho forums; dates not confirmable): an M1 MacBook Pro user on TrueSync 3.15.0 said the app "randomly starts filling up my local hard drive space" even with no folders set offline, and "the only way to resolve the issue is to disconnect the account, and then login again." — [Zoho Community: TrueSync regularly filling up my local disk](https://help.zoho.com/portal/de/community/topic/truesync-regularly-filling-up-my-local-disk?page=6)
- Earlier complaints: TrueSync broke on macOS 11 Big Sur; a user asked when it could be installed on M1 Macs "without having to make changes in the Apple Recovery Console" (kernel-extension era, pre-FPE). — [Zoho Community: True Sync not working on macOS 11](https://help.zoho.com/portal/zh/community/topic/workdrive-true-sync-not-working-on-macos-11-big-sur); [Zoho Community: TrueSync now supports macOS Ventura (beta)](https://help.zoho.com/portal/de/community/topic/workdrive-truesync-now-supports-macos-ventura-beta)
- A Finder-integration thread complains that deselecting folders for sync "makes them disappear from Finder entirely," called "a deal-breaker for our company." — [Zoho Community: Zoho Drive – Finder Integration](https://help.zoho.com/portal/en/community/topic/zoho-drive-finder-integration?page=33)
- Latest Windows TrueSync builds seen in a patch catalog: 3.14.0 (2025-06-23) and 3.15.0 (2025-07-28, "bug fix, minor release"); described as presenting WorkDrive "as a virtual drive, so files aren't downloaded until used." No equivalent macOS version/date was found. — [Adaptiva patch library: Zoho WorkDrive TrueSync x64](https://adaptiva.com/patch-library/1000006360_patch_for_zoho_workdrive_truesync_x64)
- Zoho says "WorkDrive 5.0 was recently launched" (content workflows, security, AI); no macOS TrueSync changes were tied to it in visible text. — [Zoho Community](https://help.zoho.com/portal/en/community/zoho-workdrive?page=7)

**WebDAV / API**
- No official Zoho documentation of WebDAV for WorkDrive was found. Community threads "WebDAV / FTP / SFTP protocols for syncing" and "WebDAV support" are user requests ("I believe the Zoho for Desktop app is built using a proprietary protocol... it would be really helpful if you implemented standard protocols such as WebDAV / FTP / SFTP") with no visible official reply. — [Zoho Community: WebDAV/FTP/SFTP protocols](https://help.zoho.com/portal/en-gb/community/topic/webdav-ftp-sftp-protocols-for-syncing?page=5); [Zoho Community: WebDAV support](https://help.zoho.com/portal/es/community/topic/webdav-support?page=1)
- Zoho Docs (which some WebDAV requests referenced) was discontinued 2023-03-15 in favor of WorkDrive. — [Zoho Community](https://help.zoho.com/portal/en-gb/community/topic/webdav-ftp-sftp-protocols-for-syncing?page=15)
- WorkDrive has an official REST API with OAuth 2.0 (register app in Zoho API Console, exchange client credentials for scoped access tokens); docs live at developer.zoho.com / workdrive.zoho.com/apidocs. — [Aaxonix glossary: WorkDrive API](https://aaxonix.com/resources/glossary/workdrive-api/) (third-party); [Zoho Community: official API docs announcement](https://help.zoho.com/portal/en-gb/community/topic/webdav-ftp-sftp-protocols-for-syncing?page=5)
- Developer pain points on the forum: a Node.js dev "cannot get a file upload to work correctly" (multipart `content` field confusion); another reports WorkDrive OAuth tokens "stopped working after an hour" with "the exact same template i usually use for the other zoho apps." No resolutions visible. — [Zoho Community: WorkDrive API – upload file](https://help.zoho.com/portal/de/community/topic/workdrive-api-upload-file?page=2)

**rclone `zoho` backend**
- Backend introduced in rclone v1.54; config requires `--zoho-region` matching your org's data center ("com" US/Global, "eu", "in", "com.au"; "use the same top level domain as you connect to in your browser"). — [rclone zoho.md mirror (v1.66)](https://git.frostfs.info/satyamisme/rclone/raw/tag/v1.66.0/docs/content/zoho.md)
- "Modified times are currently not supported for Zoho Workdrive" and checksums are unavailable — so rclone must fall back to size-only comparisons; a Jan 2025 forum thread asks whether this is still true (bisync warned "remote doesn't support modtime comparison"), with no answer visible. — [rclone zoho.md mirror](https://git.frostfs.info/satyamisme/rclone/src/branch/fix-4883-cache/docs/content/zoho.md); [rclone forum: Zoho modtime not supported?](https://forum.rclone.org/t/zoho-modtime-not-supported/49666)
- Throttling: rclone's lead developer (Nick Craig-Wood) said Zoho WorkDrive is "almost unusable with rclone due to rate limiting," disabled most Zoho integration tests "because of heavy throttling," and found Zoho's API-limit docs hard to locate. — [rclone forum: WorkDrive listing API calls exceed throttling limit](https://forum.rclone.org/t/rclone-invokes-more-number-of-workdrives-files-listing-api-calls-which-exceeds-the-throttling-limit/43037)
- A user reported uploads stall after ~1.4 GB ("rate limiting happens and this limit remains indefinitely"); another said "the latest release lists Zoho Workdrive as fixed, but I'm still encountering the same issues." — [rclone forum: Zoho Workdrive fixed?](https://forum.rclone.org/t/zoho-workdrive-fixed/46488)
- Fixes landed mid/late 2024: June 2024 commit removed a pre-upload existence check (a listing call Zoho throttled; "Zoho can't have duplicate files anyway"); a Sept 2024 commit (present in the v1.68.1 tree) switched uploads to Zoho's newer API response to avoid a post-upload metadata query, "to help with rclone being throttled by Zoho." — [rclone commit (frostfs mirror)](https://git.frostfs.info/KurlesHS/rclone/commit/2c13074f6c42757982c1afa1b517e5b4c5eb41aa); [rclone v1.68.1 backend/zoho](https://git.frostfs.info/satyamisme/rclone/src/tag/v1.68.1/backend/zoho)
- Zoho's own forum notes rclone's developer "respond[ed] proving that the issue is due to limits placed by Zoho" but Zoho's dev team never replied. — [Zoho Community: Zoho WorkDrive – Rclone](https://help.zoho.com/portal/en-gb/community/topic/zoho-workdrive-rclone?page=27)

**Zoho Flow**
- Zoho Flow's WorkDrive connector lists triggers "File created," "File updated," "File moved," "File renamed," "File downloaded," "File event occurred"; actions include "Create blank file," "Create folder," "Create team folder," "Create download link," "Share file or folder," "Move file or folder to trash." No "upload file from another app" action was found in Flow's listings. — [Zoho Flow: WorkDrive integrations](https://zohoflow.com/apps/zoho-workdrive/integrations)
- Zapier's WorkDrive integration does offer "New file in folder" trigger and "Upload file" action (Zapier is a separate paid product). — [Zoho KB: Zapier + WorkDrive](https://help.zoho.com/portal/en/kb/workdrive/integrations/external-integrations/zapier/articles/automate-business-workflows-with-zapier-zoho-workdrive-integration)

**QNAP HBS 3 → Zoho**
- No source mentions Zoho WorkDrive as an HBS 3 provider; QNAP's provider lists (2017 release and 2019 HybridMount launch) name Google Drive, OneDrive, Dropbox, Box, HiDrive, Yandex Disk, S3/Azure/B2/GCS, FTP/rsync/CIFS/WebDAV — never Zoho. — [QNAP 2017 HBS release](https://www.qnap.com/en-us/news/2017/qnap-officially-releases-hybrid-backup-sync-to-streamline-backup-restoration-and-synchronization); [QNAP HybridMount launch 2019](https://www.qnap.com/en/news/2019/qnap-officially-releases-file-based-cloud-gateway-hybridmount-for-elastic-hybrid-cloud-environment)
- HBS 3 can target a generic WebDAV server (Storage Spaces → Create → WebDAV; fields: name, user ID, password, server URL, proxy, validate SSL), but that is only useful if the provider exposes WebDAV — which WorkDrive does not. — [QNAP docs: Creating a storage space on WebDAV](https://docs.qnap.com/application/hybrid-backup-sync/3v21.x/en-us/creating-a-storage-space-on-webdav-63F51DE7.html)

**WorkDrive plans (for context on no-new-subscriptions constraint)**
- Third-party listings (Dec 2024 / 2026): Starter $2.50/user/mo annual (1 TB team pool, 1 GB upload limit), Team $4.50 (3 TB, 5 GB), Business $9 (5 TB, 50 GB); minimum 3 users; storage grows per added user from the 10th user. — [ZoftwareHub: Zoho WorkDrive pricing](https://zoftwarehub.com/en-ae/products/zoho-workdrive/pricing); [Capterra: Zoho WorkDrive pricing 2026](https://www.capterra.com/p/145411/Zoho/pricing/)

### Inferences
- Because TrueSync is a File Provider extension, WorkDrive appears under `~/Library/CloudStorage/` like Google Drive/OneDrive; Hazel 6.1's new "cloud-only files" folder options (see Q5) exist precisely for this class of folder. Hazel/Shortcuts can *move files into* the TrueSync folder to upload them — that is the only robust "NAS/Mac → WorkDrive" path without rclone.
- The sensible WorkDrive pipeline is: Mac inbox → Hazel rule → move to `WorkDrive/<team folder>` → TrueSync uploads. Reverse direction (WorkDrive → NAS archive) can use HBS 3 pulling from the Mac's TrueSync folder over SMB only if the files are "Make offline" (hydrated); online-only placeholders will read as zero bytes/dataless to a NAS SMB client.
- rclone-to-Zoho is usable for small, infrequent batches (`--tpslimit` low, `--transfers 1`) but not as a continuous sync engine; the lack of modtime makes `rclone sync`/`bisync` unreliable for change detection.
- Zoho Flow cannot be the mover because it lacks an upload action; it is useful only for cloud-side reactions (notify, create folders, tag records in CRM) once a file lands in WorkDrive.

### Gaps
- Exact current macOS TrueSync version number and release date (only Windows 3.15.0 / 2025-07-28 found).
- Whether the Tahoe beta build has since become the GA macOS 26 build, and what issues testers reported.
- Zoho's documented WorkDrive API rate limits (rclone's maintainer could not find them either).
- Whether rclone ≥1.69/1.70 changed Zoho modtime support (forum question unanswered).

---

## Key Question 2: Google Drive — Drive for desktop on macOS, rclone `drive`, QNAP HBS 3 native Google Drive support, Qsync, and 2025–2026 Google One / Workspace storage state

### Takeaway
Google Drive is the best-supported "personal dump" target in this stack: Drive for desktop has been native Apple Silicon since v52.0 (Oct 2021) and streams via File Provider at `~/Library/CloudStorage`; QNAP HBS 3 has a first-class Google Drive provider with one-way and two-way (including real-time) sync; rclone's `drive` backend works well if you create your own OAuth client ID. Storage policy changed on 2026-04-01: the $19.99/mo Google AI Pro plan (which replaced Google One 2 TB for subscribers) now includes 5 TB shared across Drive/Gmail/Photos.

### Cited Findings

**Drive for desktop (macOS)**
- Streaming on macOS 12.1+ uses Apple's File Provider framework; files live under `~/Library/CloudStorage`; non-downloaded files show a cloud icon; Google Drive must be enabled under Finder "Locations" or streaming won't work; the OS "can clear cached files automatically when disk space runs low." — [Google Support: Use Drive for desktop on macOS](https://support.google.com/drive/answer/12178485?hl=en)
- Stream vs. mirror: streaming keeps files mainly in the cloud and makes them available offline once opened; mirroring stores full copies locally and in the cloud; "Streaming can only be used for shared drives and files from other computers, while local folders and the desktop can only be mirrored." Switching Mirror→Stream before pending uploads finish, then deleting the old mirror folder, "can permanently lose unsynced files." — [Google Support: Stream & mirror files](https://support.google.com/drive/answer/13401938?hl=en)
- Syncing Desktop/Documents/Downloads/external or network volumes/Photos library may require granting Drive access under System Settings > Privacy & Security. — [Google Support: Use Drive for desktop on macOS](https://support.google.com/drive/answer/12178485?hl=en)
- Native Apple Silicon support arrived in Drive for desktop v52.0 (2021-10-14): "Full Apple silicon (M1) Mac support." — [9to5Google](https://9to5google.com/2021/10/22/google-drive-desktop-mac-m1-support/)
- No reliable reports of macOS 26 Tahoe-specific Drive for desktop breakage were found; Google historically did flag macOS-version compatibility (e.g., advised waiting for Sonoma 14.1 in 2023). — [Google Workspace: Drive for desktop release notes archive](https://knowledge.workspace.google.com/admin/drive/google-drive-for-desktop-release-notes-archive)
- Finder tags on Google Drive files are not propagated to other users/devices (MacScripter thread: tag visible only locally; suggested workaround is the Drive API). — [MacScripter: Duplicating Finder's file tags on Google Drive's files](https://www.macscripter.net/t/duplicating-finders-file-tags-on-google-drives-files/74207)

**rclone `drive` backend**
- rclone ships a shared default client_id; Google enforces per-client_id QPS limits (docs: default quota ~10 transactions/sec, beyond which rclone rate-limits). Docs recommend "making your own client_id" via Google API Console (enable Drive API, create OAuth client ID; any Google account can own the project). Forum answers state the default ID "is not working well these days." — [rclone drive docs (mirror commit)](https://git.frostfs.info/KurlesHS/rclone/commit/2c13074f6c42757982c1afa1b517e5b4c5eb41aa); [rclone forum: 403 rate_limit_exceeded](https://forum.rclone.org/t/error-403-rate-limit-exceeded-in-addition-i-cant-make-my-own-client-id-as-my-school-does-not-allow-that/35039/2)
- `rclone bisync` (added v1.58) is officially "an advanced command, so use with care... or data loss can result"; Google Drive users report frequent "Must run --resync to recover" aborts, "too many deletes" safety stops, and Google Docs/Sheets conversion problems; `--drive-skip-gdocs` resolved repeated resync for several users; native Docs export via `--drive-export-formats` (default docx,xlsx,pptx,svg) and import via `--drive-import-formats` (default none). — [rclone bisync command](https://rclone.org/commands/rclone_bisync/); [rclone forum: Google drive bisync nearly always requires resync](https://forum.rclone.org/t/google-drive-bisync-nearly-always-requires-resync/41276); [rclone forum: too many deletes](https://forum.rclone.org/t/failed-to-bisync-too-many-deletes-ignore-delete/31686); [rclone forum: Bisync of Googledocs](https://forum.rclone.org/t/bisync-of-googledocs-on-googledrive/29961)

**QNAP HBS 3 and Google Drive**
- QNAP FAQ (last modified 2025-12-04): HBS 3 → Storage Spaces → Create → Google Drive → OAuth sign-in → then use as backup/restore/sync destination. — [QNAP FAQ: back up to Google Drive using HBS3](https://www.qnap.com/en-ca/how-to/faq/article/how-do-i-back-up-data-to-google-drive-using-hbs3-and-restore-it)
- HBS 3 supports two-way *real-time* sync with Google Drive: HBS 3 26.1.0.723 (released 2025-08-12) fixed "an issue where after updating to HBS 3 version 26.0.0, existing two-way real-time sync jobs with Google Drive failed to sync new files to the cloud." — [QNAP Community: HBS 3 26.1.0.723 released](https://community.qnap.com/t/hbs-3-hybrid-backup-sync-26-1-0-723-released/2082)
- Google changed Drive API parameters on 2024-10-30 without notice; QNAP required HBS 3 ≥25.1.2.692 to keep Google Drive sync working. — [QNAP Community: Update HBS now to sync with Google Drive](https://community.qnap.com/t/update-hbs-now-to-sync-with-google-drive-please-note-that-google-has-updated-the-api-without-prior-notification/136)
- Security advisories give the 2025 version ladder: 25.1.4.952+ (Jan 2025, rsync CVEs) and 26.2.0.938+ (Nov 2025, Pwn2Own 2025 critical fixes; advisory revised 2026-01-03). Update via App Center on QTS/QuTS hero. — [QNAP QSA-25-46](https://www.qnap.com/zh-hk/security-advisory/qsa-25-46)
- Third-party summary of HBS 3 features: versioning ("store multiple file versions"), one-way/two-way sync; QNAP marketing: "over 20 built-in cloud services." — [MultCloud: QNAP sync software](https://www.multcloud.com/tutorials/qnap-sync-software-0121-gc.amp.html); [QNAP HBS3 landing (DE)](https://www.qnap.com/static/landing/2019/de-hbs3/index.html)

**Google One / Google AI Pro storage (2025–2026)**
- On 2026-04-01 Google raised AI Pro ($19.99/mo) storage from 2 TB to 5 TB "for all users globally at no additional cost," shared across Gmail, Drive, and Photos; the $9.99 tier remains 2 TB. — [9to5Google: Google AI Pro 5 TB](https://9to5google.com/2026/04/01/google-ai-pro-5-tb/)
- AI Pro: $19.99/mo or $199.99/yr; shareable with up to six people (five others); "if you currently pay for a Google One 2 TB plan, AI Pro replaces it. It does not stack." (guide last verified Aug 2026). — [aiera.blog: Google AI Pro in 2026](https://aiera.blog/google-ai-pro-in-2026-price-features-limits-a/)
- 2026 promo: 50% off 2 TB Premium / AI Pro annual for new subscribers ($49.99 first 12 months, then $99.99/yr); Basic 100 GB $9.99 first year, then $19.99/yr. — [Nasdaq/RTTNews](https://www.nasdaq.com/articles/google-one-cuts-2-tb-and-ai-pro-annual-plans-50-new-2026-subscribers)
- Free tier remains 15 GB shared across Gmail/Drive/Photos. — [Spliiit: Google One pricing](https://www.spliiit.com/en/blog/google-one-abonnement-stockage-prix)
- Google Workspace Business Starter = 30 GB *pooled* per user ($6/user/mo annual); Business Standard 2 TB/user, Business Plus 5 TB/user pooled; Google does not sell add-on storage for Workspace—you add users. — [Google Workspace Admin Help: storage](https://pss.support.google.com/a/answer/10071955); [9to5Google: Business Starter pooled storage (2023)](https://9to5google.com/2023/05/12/google-workspace-business-starter-pooled-storage/); [InMotion: Google Workspace storage](https://www.inmotionhosting.com/blog/google-workspace-storage/)

**Qsync (QNAP's own Dropbox-style client)**
- Mac client 5.1.6.0122 was Intel-only; community asked for an Apple Silicon build. — [QNAP Community: Qsync 5.1.6 needs Apple Silicon version](https://community.qnap.com/t/qsync-client-for-macos-v5-1-6-0122-need-new-version-for-apple-silicon/6032)
- "Qsync for Mac 6.0 Beta — open for community testing": Universal Binary (native Apple Silicon, no Rosetta 2), new backup feature, redesigned task list; requires macOS 12+, QTS/QuTS hero/QuTScloud 5.2.x+, Qsync Central 5.0.0.0+. Early tester reports: app shows "What's New" then "spinning beach ball"; install took several attempts. QNAP advises making a second copy of PC-backup data before upgrading. — [QNAP Community: Qsync for Mac 6.0 Beta](https://community.qnap.com/t/qsync-for-mac-6-0-beta-open-for-community-testing/6195); [MacG (July 2026): Qsync 6.0 sans Rosetta](https://www.macg.co/logiciels/2026/07/qsync-60-sauvegarde-desormais-les-mac-sur-les-nas-qnap-sans-rosetta-309597)
- Rosetta Check community sightings: universal v6.0.0.3002 seen; an older page still says 5.1.6 is the latest Intel-only release (conflicting/stale). — [Rosetta Check: Qsync Client](https://rosettacheck.com/apps/com.qnap.qsync)
- Reliability: Ventura iMac user reported Qsync Client would not launch (background process, "Qsync not enabled"), repeated reinstalls failed; fix advice was to purge scattered support files from Library folders. — [QNAP Community: Qsync Client won't launch on iMac](https://community.qnap.com/t/qsync-client-wont-launch-on-imac/4644)

### Inferences
- For "Google Drive as personal dump," HBS 3 one-way sync (NAS → Drive) or Drive for desktop mirroring on the Mac mini are both viable; HBS 3 two-way real-time sync works but has had regressions (26.0.0) — pin to ≥26.1.0.723 / ≥26.2.0.938 and keep versioning on.
- rclone on the Mac mini (launchd) with a personal client_id is the most flexible Drive mover; avoid bisync unless Google-native docs are excluded.
- Finder tags do not survive the trip through Google Drive (and almost certainly not WorkDrive either) — tags are a *local* routing signal only, consumed by Hazel before the file leaves the Mac.
- With Google AI Pro now 5 TB, the "personal dump" tier is unlikely to need expansion; Google Photos backups count against the same pool, so a NAS archive of originals keeps Photos from eating Drive quota.

### Gaps
- Official current HBS 3 provider list (docs.qnap.com page not reachable); only 2017/2019 lists plus "over 20" marketing claim.
- Whether Qsync for Mac 6.0 has reached GA as of Oct 2026 (sources only show beta through July 2026).
- Any Tahoe-specific Drive for desktop release notes for 2025–2026.
- Whether the user's "Workspace pro" account is Google AI Pro (consumer) or a Workspace Business edition — storage math differs.

---

## Key Question 3: iCloud Drive — can a NAS sync to it? The Mac mini as always-on hub/bridge

### Takeaway
No NAS vendor (QNAP, TrueNAS, TerraMaster) offers a native iCloud Drive target; the two real options are (a) rclone's `iclouddrive` backend (v1.69+, Jan 2025) which requires your *main* Apple ID password + 2FA (app-specific passwords rejected), 30-day trust-token re-auth, and Advanced Data Protection OFF; or (b) a Mac mini that stays logged in, with `~/Library/Mobile Documents/com~apple~CloudDocs/` synced by Hazel/rsync/HBS-over-SMB — with "Optimize Mac Storage" turned off so files are never dataless.

### Cited Findings
- rclone iCloud Drive backend: "added in v1.69" (v1.69.0 released 2025-01-12). — [rclone: iclouddrive](https://rclone.org/iclouddrive/); [rclone forum: v1.69.0 release](https://forum.rclone.org/t/rclone-v1-69-0-release/49474)
- Auth: "app-specific passwords are not accepted. Only use your regular Apple ID password and 2FA"; "ADP is currently unsupported and need to be disabled"; iPhone setting "Access iCloud Data on the Web" must be ON; "The trust token is valid for 30 days. After which you will have to reauthenticate with rclone reconnect or rclone config." — [rclone: iclouddrive](https://rclone.org/iclouddrive/); [RcloneView: add iCloud Drive](https://rcloneview.com/support/es/howto/remote-storage-connection-settings/connect-using-cli/add-icloud-drive)
- TrueNAS "lacks the ability to natively synchronise with iCloud Drive"; TerraMaster users likewise pointed to rclone as the only workaround. — [TrueNAS forum: iCloud Cloud Sync](https://forums.truenas.com/t/icloud-cloud-sync/29278); [TerraMaster forum: iCloud support request](https://forum.terra-master.com/en/viewtopic.php?p=55068)
- Mac-as-bridge path: NAS sync task pointed at `~/Library/Mobile Documents/com~apple~CloudDocs/`, one-way "upload local changes only"; "your computer needs to be running for this sync to happen." — [PicBackMan: Backup iCloud Drive to Synology NAS](https://www.picbackman.com/tips-tricks/backup-icloud-drive-to-synology-nas-in-4-easy-ways/)
- RcloneView note: on macOS the iCloud remote "reveals exactly what's stored in iCloud regardless of whether files have been downloaded locally." — [RcloneView blog: iCloud Drive cloud sync](https://rcloneview.com/support/blog/manage-icloud-drive-cloud-sync-rcloneview)
- Since macOS Sonoma, iCloud Drive uses a FileProvider-based system with two modes: with Optimize Mac Storage ON files can be evicted to dataless placeholders; OFF keeps local and cloud copies identical and blocks eviction. — [Eclectic Light: iCloud Drive in Sonoma: Optimise Mac Storage or not?](https://eclecticlight.co/2024/03/11/icloud-drive-in-sonoma-optimise-mac-storage-or-not/)
- Forcing hydration from scripts is unreliable: one guide lists `brctl download`; a third-party tool's README says Apple removed that subcommand ("It no longer works"); `fileproviderctl` subcommands (e.g. `materialize`) were missing/crashing in a Sonoma 14.4 beta; the `icloud-pull` project says reading one byte (`head -c1 <file> > /dev/null`) triggers download and dataless files read as 0 bytes. — [Ntiva: Reclaim local storage](https://support.ntiva.com/hc/en-us/articles/360056402611-Reclaim-Local-Storage-Space-by-Removing-iCloud-Drive-Downloads); [GitHub: hyper0x/icloud-pull](https://github.com/hyper0x/icloud-pull); [DroidWin: fileproviderctl not working in Sonoma](https://droidwin.com/fileproviderctl-command-is-not-working-in-sonoma/)
- Finder "Keep Downloaded" / pinning exists in the UI to prevent eviction. — [Tom's Guide: keep iCloud Drive files downloaded](https://tomsguide.com/computing/macos/how-to-keep-to-keep-icloud-drive-files-downloaded-on-your-mac)
- Mac mini hub pattern (MPU forum): users ran a Mini "backing up my NAS, running Hazel rules, storing backup copies of my Photos library"; drop-folder workflow: "on my laptop I just move files to the root of my Dropbox folder, and the Mac mini will sort them where it needs to"; Hazel can run rsync on change. — [MPU: What do you use your Mac mini for?](https://talk.macpowerusers.com/t/what-do-you-use-your-mac-mini-for/9017); [Reddit r/MacOS via redlib: folder backup to USB](https://redlib.hbubli.cc/r/MacOS/comments/qsnrp8/folder_backup_to_usb_for_macos)
- Headless gotcha: after power loss/restart "the mac returns to the login screen and none of the applications like Hazel... are available until logged in"; options are auto-login (security trade-off) or remote login via Screens/VNC; "VNC is available pre-boot but... none of your 'Login' apps can start until you've logged in, by design." — [MPU: Hosting Hazel rules on remote server](https://talk.macpowerusers.com/t/hosting-hazel-rules-on-remote-server/8252)
- Counter-view: "macOS is not a server operating system. And applications like Hazel are not meant to run as server processes" vs. "Hazel is one of the few reasons to stick with a Mac mini." — [MPU: Hazel equivalent in web app?](https://talk.macpowerusers.com/t/hazel-equivalent-in-web-app/17374)
- Hazel added native upload to remote servers in v3.1 (2013) and SFTP ecdsa/ed25519 keys in 5.1. — [MacTech: Hazel 3.1 uploads](https://www.mactech.com/2013/06/20/hazel-for-os-x-gets-the-ability-to-upload-files-to-remote-servers/amp/); [Noodlesoft: Hazel 5.1 release notes](https://www.noodlesoft.com/?p=1761)

### Inferences
- For an all-Apple household with ADP likely desirable, rclone-to-iCloud from the NAS is a poor fit (ADP must be off, monthly re-auth with the primary password). The Mac mini bridge is the pragmatic choice: it is the *only* machine in the topology that can hold TrueSync, Drive for desktop, and iCloud Drive simultaneously, so it becomes the "junction" between all three clouds and the NAS.
- On the mini: disable Optimize Mac Storage for iCloud Drive, set Drive for desktop to Mirror for the dump folder, and "Make offline" the WorkDrive folders you want the NAS to archive — otherwise the NAS will see dataless stubs over SMB.
- Mitigate the login problem with auto-login on a locked-down LAN plus FileVault off (or accept manual unlock after power events), and a UPS.

### Gaps
- No source confirms whether rclone's iCloud backend has gained ADP support or app-specific-password support since early 2025.
- No authoritative, current Apple documentation on a supported CLI to hydrate dataless iCloud files.

---

## Key Question 4: QNAP tools — HBS 3 targets, HybridMount, Qsync Client, Qfiling, Qsirch/AI Core/QuMagie, snapshots, Container Station (rclone/Syncthing/n8n/Ollama)

### Takeaway
QNAP covers the NAS-side pieces well: HBS 3 (backup/sync to Google Drive, OneDrive, Dropbox, Box, S3-class, WebDAV, rsync/SMB), HybridMount (mount a cloud as a File Station/SMB folder with two free cache licenses), Qfiling (rule-based auto-filing by name/size/date/file type with recipes and schedules), Qsirch 5.4+ (AI semantic search, needs AI Core and 8 GB RAM), snapshots (≥1 GB RAM, storage-pool based), and Container Station for Docker (n8n, Syncthing, Ollama all documented). Weak spots: Qsync's Mac client (Intel-only until the 2026 6.0 beta) and no Zoho/iCloud connectors.

### Cited Findings

**HBS 3**
- QNAP's 2017 provider list: Google Drive, OneDrive (personal + Business), Dropbox (personal + Business), Box, HiDrive, Yandex Disk, Amazon Drive, hubiC, Amazon S3/Glacier, Azure, Google Cloud Storage, Backblaze B2, HKBN, plus S3/OpenStack Swift/WebDAV-compatible services; also "another QNAP NAS or... FTP, Rsync or CIFS/SMB server." — [QNAP 2017 HBS release](https://www.qnap.com/en-us/news/2017/qnap-officially-releases-hybrid-backup-sync-to-streamline-backup-restoration-and-synchronization); [itdaily: back up cloud storage with HBS 3](https://itdaily.com/howtos/software/create-a-backup-of-your-cloud-storage-with-qnap-hbs-3)
- Generic "S3 Compatible" provider (server address + keys) used by Storj, IDrive, Impossible Cloud guides. — [Storj docs: QNAP HBS 3](https://storj.dev/dcs/third-party-tools/qnap-hybrid-backup-sync-3); [IDrive QNAP instructions](https://cloud.idrive.com/qnap-backup-instructions)
- WebDAV provider usable for two-way sync jobs (Infomaniak kDrive guide: Sync → Two-way Sync Job → WebDAV → app password; NAS must be internet-reachable). — [Infomaniak: sync kDrive and QNAP via WebDAV](https://www.infomaniak.com/en/support/faq/2405/sync-kdrive-and-qnap-via-webdav)
- 2025 versions: 25.1.4.952 (Jan 2025), 26.1.0.723 (2025-08-12), 26.2.0.938 (Nov 2025 security). — see Q2 citations.

**HybridMount**
- File Cloud Gateway mode mounts cloud storage for access via File Station and SMB/FTP/AFP/NFS/WebDAV with local caching ("near-LAN speeds"); Network Drive Mount mode mounts cloud/remote devices without caching. "One license is needed per connection that uses caching. HybridMount provides two free licenses." Launch (Nov 2019) supported "22 major file storages and object storages" incl. S3, Azure, B2, Box, Dropbox, Google Drive, OneDrive, Wasabi. — [QNAP tutorial: HybridMount](https://www.qnap.com/en-us/how-to/tutorial/article/hybridmount); [QNAP 2019 HybridMount release](https://www.qnap.com/en/news/2019/qnap-officially-releases-file-based-cloud-gateway-hybridmount-for-elastic-hybrid-cloud-environment)
- HybridMount 1.2 (2020) added mounting remote SMB/NFS server folders as local shared folders. — [QNAP 2020: HybridMount 1.2](https://www.qnap.com/en/news/2020/qnap-releases-hybridmount-1-2-enabling-users-to-locally-mount-remote-server-folders-for-easy-file-sharing)

**Qfiling**
- Three-step rule model: choose filters to find files → pick processing modules and destination → define folder structure; filters include file name, file size, modification date; "some criteria require the Qsirch app"; processing modules include subtitle matching (.srt to video folder), video transcoding to MP4, compression to ZIP/7z. — [QNAP Qfiling (DE)](https://www.qnap.com/de-de/software/qfiling)
- File-type condition exists both as a source filter and as a destination folder-structure condition (example: images → "Image" folder, documents → "Document"); recognized extension lists for Pictures (BMP, GIF, HEIC, JPG, PNG, RAW…), Music (AAC, FLAC, M4A, MP3, OGG, WAV, WMA…), Video (AVI, MKV, MOV, MP4, MPEG, WMV…). FAQ last modified 2023-12-07. — [QNAP FAQ: file extensions Qfiling recognizes](https://www.qnap.com/en/how-to/faq/con_show.php?cid=715)
- Qfiling 3.1.0 "supports all NAS models but only supports QTS 4.5.1 or later"; tutorial covers "Creating a Task" and "Creating and Editing Recipes." — [QNAP tutorial: How to use Qfiling (zh-tw)](https://www.qnap.com/zh-tw/how-to/tutorial/article/%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8-qfiling)
- Qsirch 5.0 (2020) introduced "auto-archiving files" alongside image/text-in-image search. — [QNAP 2020: Qsirch 5.0](https://www.qnap.com/en/news/2020/qnap-unveils-qsirch-5-0-enabling-image-search-text-search-in-images-and-auto-archiving-files)

**Qsirch / AI Core / QuMagie**
- Qsirch 5.4.2 (July 2024): AI semantic search ("analyze the intent and context of user queries," 23 languages), similar-image search, document preview; requires Qsirch ≥5.4.0, "8GB RAM recommended," and "QNAP AI Core must be installed in App Center." RAG search requires Qsirch ≥5.6.0. Qsirch needs QTS 5.0.1+. — [QNAP 2024: Qsirch 5.4.2](https://www.qnap.com/en/news/2024/qnap-officially-releases-qsirch-5-4-2-enhanced-ai-powered-semantic-search-and-precise-image-search-on-nas); [QNAP: Qsirch quick start guide](https://www.qnap.com/go/how-to/tutorial/article/qsirch-the-quick-start-guide)
- QNAP AI Core "primarily works in the QuMagie and QVR Face applications" (face/object recognition); Coral USB Edge TPU accelerator supported (QTS 4.5.3, AI Core 3.0.1, QuMagie 1.5.1+; up to 4 TPUs per app; lab test: 50,000 photos on TS-253D). — [QNAP product page: Coral USB Accelerator](https://www.qnap.com/en/product/g950-06809-01-qnap)

**Snapshots**
- Requirements: at least one storage pool and one volume/LUN; snapshots stored in pool free space; minimum 1 GB RAM (lowered from older 4 GB guidance); limits: 1 GB RAM → 32/NAS, 16/volume; ≥4 GB → 1024/NAS, 256/volume; QuTS hero/QES up to 65,535; x86 models on QTS 4.2+, limited ARM support from QTS 4.3.4. — [QNAP tutorial: Using Snapshots in QTS](https://www.qnap.com/en/how-to/tutorial/con_show.php?cid=285); [QNAP FAQ: How many snapshots](https://www.qnap.com/en/how-to/faq/con_show.php?cid=1572)

**Container Station workloads**
- n8n on QNAP (community walkthrough): data folder e.g. `/share/Container/n8n-data` chowned 1000:1000; Container Station → Applications → Create → paste Docker Compose; set `WEBHOOK_URL` to NAS IP:5678 and `N8N_SECURE_COOKIE=false` for local HTTP; a MyQNAP.org repo may offer n8n as an installable app. — [QNAP Community: Anyone hosted n8n on your QNAP?](https://community.qnap.com/t/anyone-ever-hosted-n8n-on-your-qnap/5166)
- n8n "Local File Trigger" node: fires on file/folder add/change/unlink; "Folder to Watch" with ignore patterns (Anymatch); self-hosted only (not n8n Cloud); "disabled by default starting in n8n 2.0 because it can pose security risks." — [n8n docs: Local File Trigger](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.localfiletrigger.md)
- Ollama on QNAP (official tutorial): Container Station 3.x+, x86_64 CPU only ("only a few ARM models are supported"), ≥8 GB RAM for 3B models / ≥16 GB for 7B, ≥20 GB free beyond model size, SSD recommended; API on port 11434, models persisted in `/share/Container/ollama`; do not expose 11434 to the internet. — [QNAP tutorial: Deploy Ollama with Container Station](https://www.qnap.com/en/how-to/tutorial/article/how-to-deploy-ollama-on-the-qnap-nas-using-container-station)
- Syncthing on QNAP: use `/share/<folder>` paths; standard compose ports 8384 (UI), 22000 TCP/UDP, 21027 UDP; "no special permissions are needed on a QNAP NAS" (vs. Synology); an old TS-210 was judged too weak. — [NASCompares Q&A](https://ask.nascompares.com/printthread.php?tid=10147); [Syncthing forum: sync macOS folders to QNAP](https://forum.syncthing.net/t/sync-mac-os-folders-to-qnap-is-it-possible/19512)
- Container Station sizing (third-party): "TS-464 with 8GB RAM the practical minimum for running 3–5 containers." — [NeedToKnowIT: Container Station setup guide](https://needtoknowit.com.au/blog/qnap-container-station-setup-guide-running-docker-on-your-nas/)

### Inferences
- Qfiling is the right NAS-side "archive sorter" for media/type/date routing (photos → `/Photos/YYYY/MM`, videos transcoded), but it is not a semantic business-vs-personal classifier; it should sit *after* the Mac-side classification step, operating on the NAS landing folder.
- HybridMount with a cached Google Drive mount lets Hazel/rclone on the Mac write to an SMB share that is really Google Drive — an alternative to running Drive for desktop on the mini — but it burns one of the two free cache licenses and adds a sync layer to debug.
- A QNAP with ≥16 GB RAM and x86 CPU can host both Ollama (classifier) and n8n (orchestrator) in Container Station; ARM models cannot.

### Gaps
- Qfiling: whether a destination can be a HybridMount/remote-NAS folder, whether Qfiling tasks run on a schedule vs. real-time, and Qfiling's current version/QTS 5.2 changes — no English source retrieved.
- QTS 5.2 / QuTS hero 5.2 release-note changes to HBS 3 or Qfiling specifically — only security-advisory version numbers found.
- Current official HBS 3 provider list page.

---

## Key Question 5: Rule engines & classification — Hazel 6, Folder Actions, Shortcuts automations (macOS 26), launchd, n8n/Node-RED on the NAS, and inserting a local LLM (Ollama / Apple "Use Model")

### Takeaway
Hazel 6.1 ($42 one-time, free update for 6.x owners, macOS 13+, Tahoe-ready with new cloud-only-file options, built-in OCR, "Run Shortcut" and shell-script actions) remains the most complete Mac rule engine; macOS 26 Tahoe added native Shortcuts folder automations (Added/Modified/Removed, "Run Immediately" or after confirmation) and a "Use Model" action for on-device/Private Cloud Compute inference, which together can approximate a free LLM classifier. launchd WatchPaths and AppleScript Folder Actions still exist but are cruder. On the NAS, n8n's Local File Trigger + Ollama API provide a Mac-independent classifier/orchestrator.

### Cited Findings

**Hazel 6.x**
- Pricing: "$42 (USD) for a single user license while the family pack is $65"; upgrade from any previous version $20; license is perpetual for the major version purchased ("Even if a new major version comes out, your license will still unlock the version you purchased"). Hazel 6.0 required macOS 12+; supports all Apple Silicon Macs. — [Noodlesoft: Hazel 6 FAQ](https://www.noodlesoft.com/?p=1852)
- Per a Japanese review: single license covers up to two Macs; Family Pack five people/Macs; current release 6.1.2 requires macOS 13+. — [monodeck.jp](https://monodeck.jp/?p=5918)
- Hazel 6 features: "on-the-fly text recognition" (OCR for images and PDFs without text layer), revert files, read encrypted PDFs, custom list attributes, locked-file handling, pause action. — [Noodlesoft: Hazel 6 FAQ](https://www.noodlesoft.com/?p=1852); [ifun.de: Hazel 6.0](https://www.ifun.de/automatisierung-hazel-app-fuer-macos-erscheint-in-version-6-0-242226/)
- Hazel 6.0.2 added option to use text recognition "even if the file already has searchable text"; "as needed" remains default. — [Noodlesoft: Hazel 6.0.2 release notes](https://www.noodlesoft.com/?p=1894)
- "Contents contain" relies on Spotlight but falls back to "contains match" (reads file contents; slower, more accurate) when Spotlight fails. — [MPU forum: Hazel not reading PDF](https://talk.macpowerusers.com/t/hazel-not-reading-pdf/21862)
- Hazel 6.1 (Tahoe update): Liquid Glass UI, Tahoe menu icons, "New folder options to tell Hazel how do deal with cloud-only files," workaround for Apple bug showing Hazel's icon on external drives in Finder sidebar (recommendation: disable Hazel's Finder extension if top-level drive items misbehave), preview runs in background; requires macOS 13+; $42 new / free update. — [Noodlesoft: Hazel 6.1 release notes](https://www.noodlesoft.com/?p=2008); [TidBITS: Hazel 6.1](https://talk.tidbits.com/t/hazel-6-1/32328)
- Shortcuts integration since Hazel 5.1 (Nov 2021): "Added support for running Shortcuts on Monterey" (Run Shortcut action); AppleScript scriptability of Hazel's run/rule status. — [Noodlesoft: Hazel 5.1 release notes](https://www.noodlesoft.com/?p=1761); [MacSparky: Hazel 5.1](https://www.macsparky.com/blog/2021/11/hazel-51/)
- A Noodlesoft forum thread titled "Hazel no longer lists Shortcuts" exists (contents not retrievable) — suggests the Shortcuts picker has been flaky at times. — [Noodlesoft forums](https://www.noodlesoft.com/forums/viewforum.php?f=4)
- Hazel turned 20 in 2026; Noodlesoft ran a 35% anniversary sale ($27.30 single). — [MacSparky: Hazel Turns 20](https://www.macsparky.com/?p=19684)
- Hazel 5.3.4 was the last 5.x seen (Apr 2024). — [Softpedia: Noodlesoft](https://mac.softpedia.com/publisher/Noodlesoft-12432.html)

**macOS 26 Tahoe Shortcuts automations**
- Tahoe adds Shortcuts automations triggered by filesystem changes: choose a folder, pick "Added, Modified, or Removed"; option "Ignore subfolders"; "More than two decades after Folder Actions... Apple has built a modern take on the feature." — [Six Colors: Get started with folder automation in macOS Tahoe](https://sixcolors.com/post/2025/08/get-started-with-folder-automation-in-macos-tahoe/)
- Run mode: "Choose Run Immediately or Run After Confirmation (notification prompt)." — [MacMost: Introduction to Shortcuts Automation in macOS Tahoe](https://macmost.com/an-introduction-to-shortcuts-automation-in-macos-tahoe.html); [AppleWorld.Today: create automations in macOS Tahoe](https://appleworld.today/2025/10/how-to-create-automations-in-macos-tahoe/)
- "Use Model" action: Apple's description — "generate responses that feed into the rest of their shortcut using on-device models or Private Cloud Compute." — [MacScripter: Changes to the Shortcuts app in Tahoe](https://www.macscripter.net/t/changes-to-the-shortcuts-app-in-tahoe/77180)
- Jason Snell floated the concept of a folder-triggered shortcut that processes a file with on-device AI; no step-by-step walkthrough combining folder trigger + Use Model was found. — [Six Colors](https://sixcolors.com/?p=37562)

**Folder Actions / launchd**
- Folder Actions Setup is reached via Finder → Services → Folder Actions Setup; since Mojave it is a "restricted system service" requiring a manual "Run Service" confirmation that "can't be automated." — [macosxautomation.com: Folder Actions Setup](https://www.macosxautomation.com/automator/folder-action/folder-actions-setup.html); [Macworld: attach action to Mac folder](https://www.macworld.com/article/634271/attach-action-mac-folder.html)
- An AppleScript folder action can call a Shortcut directly ("wired directly into AppleScript"). — [Matthew Cassinelli: Shortcuts and macOS Folder Actions](https://matthewcassinelli.com/automation-shortcuts-macos-folder-actions)
- launchd `WatchPaths`: fires a job when the path changes, but only the top level of a directory is watched (not subdirectories) and the script is not told *which* file changed — it must diff listings itself; Apple DTS says jobs that never fire are usually blocked by Login Items & Extensions privileges. — [Apple Discussions: Launchd WatchPath arguments](https://discussions.apple.com/thread/348077?page=1); [Apple Developer Forums: Launch Agent trigger on folder changes](https://developer.apple.com/forums/thread/788105); [DEV: trigger any action when a file changes on macOS](https://dev.to/mayeu/how-to-trigger-any-action-when-a-file-or-folder-changes-on-macos-on-the-cheap-3dcp)

**NAS-side orchestration & LLM**
- n8n Local File Trigger and Ollama-on-QNAP details: see Q4 citations ([n8n docs](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.localfiletrigger.md); [QNAP Ollama tutorial](https://www.qnap.com/en/how-to/tutorial/article/how-to-deploy-ollama-on-the-qnap-nas-using-container-station)).
- Hazelnut: a terminal-based Hazel-like organizer (glob/regex, extension lists, size, age; move/copy/rename/trash/archive/run command) — a candidate for running rules on Linux/NAS. — [Terminal Trove: hazelnut](https://terminaltrove.com/hazelnut/)
- No published source was found describing Hazel → shell script → Ollama classification; the search returned only general Hazel material. — [MakeTechEasier: custom Hazel rules](https://maketecheasier.com/hazel-rules-automate-file-management)

### Inferences
- Classification pipeline that fits the constraints: Hazel watches `~/Inbox`; rule 1 cheap heuristics (name patterns, Finder tag `biz`/`home`, kind, sender folder); rule 2 fallback "Run shell script" that extracts text (Hazel's OCR'd text isn't exposed to scripts, so use `mdls`/`pdftotext`/Vision via a Shortcut) and POSTs to `http://nas:11434/api/generate` with a prompt like "Answer BUSINESS or PERSONAL"; script writes a Finder tag; rule 3 moves tagged files to WorkDrive / Google Drive mirror / NAS SMB share. On Tahoe, the same can be prototyped free with a folder automation + "Use Model" + "Set File Tags"/"Move File," at the cost of weaker rule logic and less observability than Hazel.
- Hazel's 6.1 cloud-only-file handling matters because the destination folders (TrueSync, Drive streaming) are File Provider folders; set Hazel to not download/evaluate cloud-only placeholders in those folders.
- launchd/Folder Actions are reasonable only for a single flat inbox with a script that does its own diffing; Hazel's rule debugger and nested-folder support make it worth the one-time $42.

### Gaps
- No documented example of "Use Model" output being parsed for routing decisions, nor its throughput/limits (search budget exhausted before this query).
- No evidence either way on whether Hazel can pass recognized OCR text to a script action.
- Node-RED on QNAP was not researched (budget); n8n covers the same role.

---

## Key Question 6: Practical separation strategy — inbox folders, Finder tags, naming conventions, separate macOS accounts/Focus, keeping business docs out of iCloud and personal photos out of WorkDrive, "Desktop & Documents in iCloud" pitfalls

### Takeaway
Apple and every major sync vendor say only one service may own Desktop & Documents; the safe pattern is to leave Desktop & Documents out of iCloud (or let iCloud own them and never point Drive/Zoho at them), create explicit per-device inbox folders, and treat Finder tags as a local-only routing signal because they don't survive Google Drive (and likely WorkDrive) sync.

### Cited Findings
- Apple: before enabling iCloud Desktop & Documents, "turn off any other third-party app or cloud service that syncs your Desktop or Documents folder. Syncing by other third-party apps or cloud services can interfere with or stop the updating performed by iCloud Drive." — [Apple Support: Mac Help (macOS 12 guide)](https://support.apple.com/en-kz/guide/mac-help/mchl1a02d711/12.0/mac)
- Dropbox moves Desktop/Documents inside its own folder when its backup feature is on; running two providers on the same folders "can cause file conflicts, lower performance, and possibly data loss"; release them via Backups → Manage Backups → Change folders, choosing "Keep content in folders on this Mac." — [Dropbox forum](https://dropboxforum.com/discussions/101001016/dropbox-backup-has-my-movies-music-and-downloads-folders-on-my-mac-so-no-icloud-/706306/replies/706544)
- Google Drive for desktop also offers to back up Desktop/Documents (per-folder "Sync with Google Drive" toggle → "Stop syncing"); OneDrive does so by default on some managed Macs; university IT advises disabling it in all but one app. — [Syracuse University IT wiki](https://answers.atlassian.syr.edu/wiki/x/pj-qC)
- Google's own docs: local folders (Desktop etc.) "can only be mirrored," and syncing Desktop/Documents requires Privacy & Security permission. — [Google Support: Stream & mirror](https://support.google.com/drive/answer/13401938?hl=en)
- iCloud Drive conflict handling is weak per a vendor blog ("silently picks a version"); promotional source. — [Cloud Cub (Notion)](https://mitado.notion.site/Cloud-Cub-Protect-Your-Files-from-Silent-Sync-Disasters-1b8932dcbc8480f59a25e698f8b089a2)
- Finder tags on Google Drive files are not visible to other users/devices. — [MacScripter](https://www.macscripter.net/t/duplicating-finders-file-tags-on-google-drives-files/74207)
- TrueSync default is online-only; files placed in an offline folder become offline — so an "offline" WorkDrive inbox folder is a reliable upload target. — [Zoho TrueSync for Mac](https://help.zoho.com/portal/en/kb/workdrive/desktop-apps/workdrive-truesync/articles/truesync-for-mac)
- Drive for desktop v52.0 added Photos Library upload support including iCloud-synced media — i.e., Google Photos backup from a Mac can pull iCloud Photos content into Google storage. — [9to5Google](https://9to5google.com/2021/10/22/google-drive-desktop-mac-m1-support/)
- Google AI Pro / One storage is shared across Drive, Gmail, and Photos. — [9to5Google: AI Pro 5 TB](https://9to5google.com/2026/04/01/google-ai-pro-5-tb/)
- Drop-folder pattern in practice: "on my laptop I just move files to the root of my Dropbox folder, and the Mac mini will sort them." — [MPU forum](https://talk.macpowerusers.com/t/what-do-you-use-your-mac-mini-for/9017)

### Inferences
- Recommended layout: `~/Inbox` (local, not in iCloud) on each Mac, synced to the mini by Syncthing or iCloud Drive (a dedicated `iCloud Drive/Inbox` is acceptable since iCloud is the one service allowed to touch everyday Apple folders); the mini's Hazel routes out of the inbox so business files never *rest* in iCloud longer than minutes.
- Keep WorkDrive folders "online-only" on laptops (so personal photo dumps can't accidentally land there) and only "offline" on the mini.
- Naming convention `YYYY-MM-DD_<client|home>_<desc>` plus tags `biz`/`home` gives Hazel two cheap signals before any OCR/LLM step; tags must be applied *before* the file crosses into a cloud folder.
- Separate macOS user accounts (work vs. personal, each signed into a different iCloud account) are the strongest separation, but they double the Hazel/Drive/TrueSync configuration and Hazel's single license covers two Macs, not necessarily two users on a headless server. Not verified by a source (search budget exhausted).
- Photos: let iCloud Photos be canonical, archive originals to the NAS (QuMagie/Qfiling by date), and *do not* enable Drive for desktop's Photos Library upload on the mini unless you want Google Photos duplicates eating the 5 TB pool.

### Gaps
- No source retrieved on macOS Focus modes or Fast User Switching as a separation tool (search budget exhausted).
- No Apple documentation retrieved for the current (macOS 26) Desktop & Documents iCloud setting UI; the citation is the macOS 12 guide.

---

## Key Question 7: 2025–2026 changes — macOS 26 Tahoe (Shortcuts automations, "Use Model"), File Provider changes affecting Google Drive/Zoho, QTS 5.2 / QuTS hero updates to HBS 3 or Qfiling

### Takeaway
The material 2025–2026 changes are: macOS 26 Tahoe's Shortcuts folder automations and "Use Model" action (Aug–Sept 2025); Hazel 6.1's Tahoe compatibility release; Zoho's TrueSync Tahoe beta; QNAP's Qsync for Mac 6.0 Universal beta (2026) requiring QTS/QuTS hero 5.2.x; HBS 3 26.x releases (Aug and Nov 2025) with a Google Drive two-way real-time sync fix and critical security fixes; rclone's iCloud Drive backend (Jan 2025); and Google's AI Pro 5 TB bump (April 2026). No File Provider API break affecting Google Drive or TrueSync in Tahoe was found.

### Cited Findings
- Tahoe Shortcuts folder automations and "Use Model": see Q5 ([Six Colors](https://sixcolors.com/post/2025/08/get-started-with-folder-automation-in-macos-tahoe/); [MacScripter](https://www.macscripter.net/t/changes-to-the-shortcuts-app-in-tahoe/77180); [MacMost](https://macmost.com/an-introduction-to-shortcuts-automation-in-macos-tahoe.html)).
- Hazel 6.1 for Tahoe (glass UI, cloud-only-file options, Finder-sidebar bug workaround). — [Noodlesoft: Hazel 6.1 release notes](https://www.noodlesoft.com/?p=2008)
- Zoho TrueSync macOS 26 beta build. — [Zoho Community](https://help.zoho.com/portal/en/community/topic/workdrive-truesync-for-macos-26-tahoe-beta)
- Qsync for Mac 6.0 beta: Universal Binary, backup feature; requires QTS/QuTS hero/QuTScloud 5.2.x+ and Qsync Central 5.0.0.0+. — [QNAP Community](https://community.qnap.com/t/qsync-for-mac-6-0-beta-open-for-community-testing/6195); [MacG, July 2026](https://www.macg.co/logiciels/2026/07/qsync-60-sauvegarde-desormais-les-mac-sur-les-nas-qnap-sans-rosetta-309597)
- HBS 3 26.1.0.723 (2025-08-12) Google Drive two-way real-time fix; 26.2.0.938 (Nov 2025) Pwn2Own critical fixes; 25.1.4.952 (Jan 2025) rsync CVE fixes. — [QNAP Community](https://community.qnap.com/t/hbs-3-hybrid-backup-sync-26-1-0-723-released/2082); [QNAP QSA-25-46](https://www.qnap.com/zh-hk/security-advisory/qsa-25-46)
- Google Drive API parameter change 2024-10-30 forced HBS 3 ≥25.1.2.692. — [QNAP Community](https://community.qnap.com/t/update-hbs-now-to-sync-with-google-drive-please-note-that-google-has-updated-the-api-without-prior-notification/136)
- rclone v1.69.0 (2025-01-12) added iCloud Drive backend. — [rclone forum](https://forum.rclone.org/t/rclone-v1-69-0-release/49474); [rclone: iclouddrive](https://rclone.org/iclouddrive/)
- Google AI Pro 5 TB from 2026-04-01. — [9to5Google](https://9to5google.com/2026/04/01/google-ai-pro-5-tb/)
- Qsirch 5.4.2 (July 2024) AI semantic search; Qsirch 5.6 RAG search. — [QNAP 2024](https://www.qnap.com/en/news/2024/qnap-officially-releases-qsirch-5-4-2-enhanced-ai-powered-semantic-search-and-precise-image-search-on-nas); [QNAP Qsirch quick start](https://www.qnap.com/go/how-to/tutorial/article/qsirch-the-quick-start-guide)
- NetDrive (third-party cloud mounter) certified Tahoe compatibility on 2026-04-01 — illustrative that third-party mounters needed explicit Tahoe work. — [NetDrive blog](https://www.netdrive.net/blog/netdrive-on-macos-26-tahoe/)
- n8n 2.0 disabled Local File Trigger by default. — [n8n docs](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.localfiletrigger.md)

### Inferences
- Nothing found suggests Tahoe broke File Provider–based clients; both Google (already FP since macOS 12.1) and Zoho (FPE redesign) shipped/tested Tahoe builds. The Hazel 6.1 note about Finder-sidebar icon bugs is the only Tahoe-specific filesystem quirk surfaced.
- Keep HBS 3 at ≥26.2.0.938 both for the Google Drive fix and for the critical security patches; keep QTS on 5.2.x to be eligible for Qsync 6.0 when it leaves beta.

### Gaps
- No QTS 5.2 / QuTS hero 5.2 release notes naming HBS 3 or Qfiling feature changes were retrieved.
- No Google Drive for desktop 2025–2026 release notes retrieved.
- No details on "Use Model" quotas, supported models, or whether it runs unattended in a "Run Immediately" folder automation (the obvious prerequisite for headless classification).

---

## Summary pipeline sketch (synthesis for the report writer; inferences unless cited above)

1. **Capture** — each Mac/iPhone drops into one inbox: `iCloud Drive/Inbox` (fine for personal + transit) or a local `~/Inbox` synced to the mini via Syncthing (keeps business files out of iCloud entirely).
2. **Classify on the Mac mini (always-on, auto-login, Optimize Mac Storage off)** — Hazel 6.1 ($42): tag/name heuristics → OCR "contents contain" → fallback shell script calling Ollama on the NAS (port 11434) or a Tahoe Shortcut "Use Model"; result written as a Finder tag.
3. **Route** — Hazel moves: `biz` → `WorkDrive/<team folder>` (TrueSync, folder set "Make offline"); `home` → `Google Drive/Dump` (Drive for desktop, Mirror mode); everything → `smb://nas/Archive/Inbox` (rsync or Hazel upload) for retention.
4. **NAS post-processing** — Qfiling recipes sort `/Archive/Inbox` by type/date; snapshots protect against sync-propagated deletes; HBS 3 (≥26.2.0.938) one-way backs up chosen NAS folders to Google Drive, and optionally pulls the mini's Drive/WorkDrive offline folders over SMB for a second copy.
5. **Do not** — point HBS 3 at Zoho (no provider, no WebDAV), run rclone↔Zoho continuously (throttling, no modtime), run rclone↔iCloud with ADP on (unsupported), let two apps own Desktop & Documents, or expect Finder tags to survive any cloud hop.
