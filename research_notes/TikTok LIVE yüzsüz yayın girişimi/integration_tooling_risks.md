# TikTok LIVE event integration (comments, gifts, likes, follows, shares, joins) into games and OBS: tooling, architecture and risks (status as of 2026-11-21)

Research notes for the report writer. Scope: how a Windows 11 PC creator can read TikTok LIVE events in real time and feed them into a custom game (UE5, Unity, Godot or a browser overlay) and OBS Studio, plus how to send video to TikTok and what the account and ToS risks are.

Method caveat: eulerstream.com, tikfinity.zerody.one, developers.tiktok.com and streamer.guide could not be fetched directly from this research session (DNS blocked by the sandbox). Facts about them come from search-result snippets, GitHub READMEs, or the npm/PyPI registries, which were reachable. Package version dates come straight from registry.npmjs.org and pypi.org JSON (queried 2026-11-21).

---

## 1. Official TikTok developer offerings: is there an official API for reading LIVE events, and is there an official path for an individual developer in Turkey?

### Takeaway
No. As of late 2026 I found no official TikTok API, webhook or SDK that lets third parties read LIVE comments, gifts, likes, follows or joins. TikTok's public developer products (Login Kit, Display API, Content Posting API, Research API, Mini Games SDK, Effect House) cover login, public video data, video posting, restricted academic research, in-app mini games and AR/gift *design*. None of them exposes a LIVE event stream to an individual developer in Turkey or anywhere else. Every working route is unofficial: reverse-engineered or third-party.

### Cited Findings
- The TikTokLive (Python) README states that TikTok does not offer an official API for reading livestream events and calls itself "an unofficial, open-source TikTok LIVE API client" — [isaackogan/TikTokLive on GitHub](https://github.com/isaackogan/TikTokLive)
- Streamer.bot's official FAQ: "TikTok support will be considered if/when they release a usable public API. TikTok API support is unknown at this time, but a Streamer.bot integration is available by TikFinity." — [Streamer.bot docs: Platform Support](https://docs.streamer.bot/faq/platform-support)
- A developer write-up titled "There Is No Official TikTok LIVE API" says the TikTok Business API covers only ads and shop analytics, not LIVE interactions. The author discloses that he builds a competing API, so the source is biased, but this point matches every other source. — [DEV Community](https://dev.to/rngrow_26ac7d2e4c5dc10a00/there-is-no-official-tiktok-live-api-here-is-how-to-get-the-data-anyway-62n)
- Login Kit, Display API and Content Posting API are linked products. Content Posting API and Display API both need Login Kit plus product approval. Display API reads public content and Content Posting API uploads and publishes videos. None of them covers LIVE events. — [TikTok for Developers: Login Kit](https://developers.tiktok.com/products/login-kit); [Display API get started](https://developers.tiktok.com/docs/en/display-api-get-started); [Login Kit overview](https://developers.tiktok.com/docs/en/login-kit-overview)
- The Research API is limited to non-profit academic and independent research institutions in the US, EU/EEA, UK, Switzerland and Norway/Iceland/Liechtenstein (beta), plus some Brazilian youth-safety researchers. Access requires a research proposal, independence from commercial interests and non-commercial use. Turkey is not listed. — [TikTok Research Tools product page](https://developers.tiktok.com/products/research-api); [Research API FAQ](https://developers.tiktok.com/doc/research-api-faq); [TechCrunch, 2023](https://techcrunch.com/2023/02/21/tiktok-expands-its-research-api-to-nonprofit-academic-institutions-in-the-u-s)
- A 2025 third-party report describes a non-profit research institute that was not a university being refused, or offered only limited access. — [KInIT, "When Research APIs Close the Door"](https://kinit.sk/?p=39790)
- Mini Games SDK: the official changelog shows SDK 0.4.0 / Version 43.0.0 (December 4, 2025) added `TTMinis.game.connectSocket` for websocket connections. I found nothing linking it to LIVE gift or comment events. — [TikTok for Developers: Mini Games SDK Changelog](https://developers.tiktok.com/doc/mini-games-base-library-update-log?enter_method=left_navigation)
- Effect House offers a "TikTok LIVE Gift" program for designing interactive LIVE gift effects (the "LIVE Interactive Gift Effect Challenge" has ended and winners were listed; entrants were told to use Effect House v4.24). The Giftopia program rewards designers of animated and interactive LIVE Gifts. This is about designing gifts, not reading gift events into an outside app. — [Effect House: LIVE Interactive Gift Effect Challenge](https://effecthouse.tiktok.com/latest/live-interactive-gift-effect-challenge); [Effect House: TikTok LIVE Gift](https://effecthouse.tiktok.com/live-gift?enter_method=top_navigation)
- An older Effect House community feature request asked for a node that detects gifts and for using one's own effects on LIVE, which suggests this did not exist when it was posted (around 2023). I found no current documentation confirming a gift-detection node. — [Effect House Community feature request](https://community.effecthouse.com/t/i-would-love-to-use-my-own-effects-on-tiktok-live/8572)
- I found no official TikTok "LIVE Studio developer" or interactive-games partner program in search results. Every interactive-game platform that turned up (TikLive Games, Livecade, Live to Gift, interactivelivegames.com, Tikqo, TikPlay) is a third party, and Live to Gift's "Partnership Program" is a referral scheme. — [tiklivegames.io](https://tiklivegames.io/studio); [Livecade](https://livecade.io/); [Live to Gift](https://livetogift.com/); [interactivelivegames.com](https://interactivelivegames.com/en); [Tikqo](https://tikqo.com/en/games); [TikPlay](https://tikplay.games/en)

### Inferences
- For an individual developer in Turkey there is no official, sanctioned path to read LIVE gifts and comments in 2026. The Research API is the only TikTok data product with eligibility rules, and it excludes Turkey and commercial use; it also does not cover LIVE event streams.
- Any "TikTok LIVE interactive game" on the market today runs on reverse-engineered webcast access, either directly (open-source libraries) or through an intermediary (Euler Stream, TikFinity, tik.tools and similar).
- Effect House LIVE gifts and the Mini Games SDK are worth watching. If TikTok ever opens an official LIVE interaction API, those are the most likely places, but nothing usable exists today.

### Gaps
- developers.tiktok.com could not be fetched directly, so I could not read the full current product list or changelog (search snippets show it was updated August 19 and 26, 2026). A LIVE-related product added recently may have been missed.
- I could not confirm whether the official Mini Games websocket can receive LIVE gift events, or how it authenticates.
- I did not check whether Douyin's (China) official interactive-LIVE open platform has any global TikTok equivalent; I found no source either way.

---

## 2. Unofficial libraries: status in 2026, signing server (Euler Stream), rate limits, breakages, transport and event types

### Takeaway
The two reference libraries, TikTok-Live-Connector (Node.js/TypeScript) and TikTokLive (Python), are actively maintained in 2026. Both had releases in September 2026 and are coordinated through Isaac Kogan, who also runs the Euler Stream signing service. Both connect over a WebSocket to TikTok's internal "Webcast" push service, need a signed token from Euler Stream (free community tier, paid tiers from $50/month), and expose 50 to 90+ event types, including gift streaks with `repeatCount`/`repeatEnd`. Both describe themselves as reverse-engineering projects that "TikTok can change without notice." The C# (TikTokLiveSharp/TikTokLiveUnity), Java and Go ports exist but are less clearly maintained.

### Cited Findings

**TikTok-Live-Connector (Node.js / TypeScript)**
- npm `tiktok-live-connector`: latest is 2.5.0, published 2026-09-16. Earlier releases were 2.4.0 (2026-06-15), 2.4.1 to 2.4.3 (2026-07-16 to 2026-07-22) and 2.4.4 (2026-08-18). License in package metadata is AGPL-3.0-only (the README says "modified AGPL"). Dependencies include `tiktok-live-api-sdk` 0.4.0-beta1 and `tiktok-live-proto`. — [npm registry: tiktok-live-connector](https://registry.npmjs.org/tiktok-live-connector) (queried 2026-11-21); [npm page](https://npmjs.com/package/tiktok-live-connector)
- The repo stays under `zerodytrash`. The README credits Zerody (original reverse engineering and protobuf decoding), Isaac Kogan (TypeScript rewrite and sign-server maintainer) and sleepylessons (docs). It warns: "This is **not** a production-ready API. It is a reverse engineering project" and "Unofficial, Unaffiliated with ByteDance Ltd." — [zerodytrash/TikTok-Live-Connector](https://github.com/zerodytrash/TikTok-Live-Connector)
- Transport and signing, from the README FAQ: "Connecting requires a signed WebSocket token, provided by Euler Stream with free community rate limits; an API key raises those limits." The library reads TikTok's internal Webcast push service over WebSocket. Polling is used only for `waitUntilLive`, at a minimum interval of 30 seconds. — [npm README](https://registry.npmjs.org/tiktok-live-connector); [GitHub](https://github.com/zerodytrash/TikTok-Live-Connector)
- Connection options include `signApiKey` (Euler key), `enableExtendedGiftInfo` (fetches the room gift list so gift events carry name, cost and image), `processInitialData` (emits recent chat history from the sign response), `useMobile` (mobile WS flow, needs an authenticated session) and proxy options. `fetchRoomId`/`fetchIsLive` fall back from HTML scraping to the TikTok API to Euler Stream. — [npm README](https://registry.npmjs.org/tiktok-live-connector)
- Event types listed include chat, member (joins), like, social, follow, share, roomUser (viewer count), emote, questionNew, gift, envelope (treasure chest: `diamondCount`, `peopleCount`), giftBroadcast, goalUpdate, superFan/superFanJoin/subNotify/superFanBox, linkMicBattle, linkMicArmies, linkMicBattleTask, linkMicBattlePunish, pollMessage, rankUpdate, captionMessage, imDelete, streamEnd and others. — [GitHub README](https://github.com/zerodytrash/TikTok-Live-Connector); [npm README](https://registry.npmjs.org/tiktok-live-connector)
- Gift streak semantics, quoted from the README: streakable gifts (`giftType: 1`) "are triggered again and again with an increased `repeatCount` value… after the end of the streak, another gift event is triggered, which signals the end of the streak via `repeatEnd: true`… even if the user sends a `giftType: 1` gift only once, you will receive the event twice. Once with `repeatEnd: false` and once with `repeatEnd: true`." — [npm README](https://registry.npmjs.org/tiktok-live-connector)
- Sending chat is available again in 2.x: `sendMessage(content)` "requires a valid Euler Stream API key and an authenticated session bundle (cookies or OAuth token)". An older 2.0.0 note said `sendMessage` had been removed, so this has changed over time. — [npm README 2.5.0](https://registry.npmjs.org/tiktok-live-connector); older note via [socket.dev package overview](https://socket.dev/npm/package/tiktok-live-connector/overview/1.1.7)

**TikTokLive (Python)**
- PyPI `TikTokLive`: latest is 7.0.1, uploaded 2026-09-10. 7.0.0 came out 2026-08-18, with a 6.6.6 back-port on 2026-07-21; 6.6.5 was 2025-10-08 and 6.6.1 was 2025-08-09. — [PyPI JSON: TikTokLive](https://pypi.org/pypi/TikTokLive/json) (queried 2026-11-21)
- README: "Connect to any TikTok LIVE stream… using just a creator's @unique_id. No login, no credentials or app are required." Note: "This is not a production-ready API. It is a reverse engineering project. Use the WebSocket API… by Euler Stream for production." — [PyPI TikTokLive](https://pypi.org/project/TikTokLive/); [GitHub](https://github.com/isaackogan/TikTokLive)
- Isaac Kogan is "Creator, Primary Maintainer, and Reverse-Engineering". The repo has about 1.7k stars and 914 commits. Its signature server defaults to Euler Stream and can be swapped. `tiktok_sign_api_key` is "A Euler Stream API key used to increase rate limits." Proxies "may be subject to reduced connection limits at times of high load". — [GitHub](https://github.com/isaackogan/TikTokLive)
- v7 change: events are auto-generated from the TikTokLiveProto v3 schema, and "if you don't see one you used to rely on, it's because TikTok removed it from the schema." Gift fields were renamed (`name`, `type`, `image`) with aliases kept. New events include SuperFanJoinEvent, SuperFanBoxEvent and LinkmicBattleTaskEvent. The README cites "50+" event types: GiftEvent, LikeEvent, CommentEvent, FollowEvent, ShareEvent, RoomUserSeqEvent, LinkMic battle events, PollEvent, CaptionEvent, LiveEndEvent and others. — [GitHub](https://github.com/isaackogan/TikTokLive)
- Python streak handling: "streakable gifts trigger multiple GiftEvents… with event.repeat_count incrementing each time. The final gift in a streak carries event.repeat_end == 1. Use the event.streaking helper to filter intermediate events out and only act on the final one." — [PyPI TikTokLive README](https://pypi.org/project/TikTokLive/)
- The README has a section on ports to "Node.js, Java, C#, Go, Rust". — [PyPI TikTokLive README](https://pypi.org/project/TikTokLive/)

**C# / Unity port**
- TikTokLiveSharp (frankvHoof93) is at README version v1.2.2, with 215 commits, 39 open issues, no published GitHub releases and no release date. It is not archived. It ships TikTokLiveUnity with its own Unity setup guide and uses an older protobuf-net for Unity compatibility. It warns that the message schema was completely replaced, which "can lead to breaking changes". — [frankvHoof93/TikTokLiveSharp](https://github.com/frankvHoof93/TikTokLiveSharp)
- On NuGet, the `tiktoklivesharp` package ID lists versions only up to 0.1.4, so the v1.2.2 code may not be published under that ID. — [NuGet flat container index](https://api.nuget.org/v3-flatcontainer/tiktoklivesharp/index.json) (queried 2026-11-21)

**Euler Stream (the signing and managed-WebSocket service behind the libraries)**
- Plans per the official SDK README ("Updated for 2026"):
  - Community: free forever, 2,500 requests/day, 25 cloud WebSockets, 5 TikTok LIVE alerts.
  - Business: $50/month with a 48-hour trial; 10,000 requests/day, 100 WebSockets, 50 alerts.
  - Enterprise: custom; 250,000+ requests/day, 1,500+ WebSockets.
  - Add-ons such as premium webcast routes, signature routes and CAPTCHA routes are paid-plan only.
  - API key: free signup, "no credit card required", key taken from the dashboard.
  - It is "an independent, unofficial third-party project… not affiliated with, endorsed by, or connected to TikTok or ByteDance." — [eulerstream/euler-api-sdk on GitHub](https://github.com/eulerstream/euler-api-sdk)
- One hosted demo page claims the free tier needs no API key. That contradicts the SDK docs and is unverified. — [tiktokliveapi.pages.dev](https://tiktokliveapi.pages.dev/)
- A competitor's troubleshooting page describes a sign-service error where "too many [connections were] started in a short time", which is a rate-limit failure mode. — [tik.tools troubleshooting](https://tik.tools/guides/troubleshooting)

**Commercial managed alternatives (unofficial, not affiliated with TikTok)**
- tik.tools / TikTool: managed `wss://api.tik.tools`, JSON `{event, data}` messages, "90+ event types", Node/Python SDKs, and HMAC-signed webhooks (`live.start`, `live.end`). It advertises a free key, with a claimed 5,000 requests/day per its own comparison pages, which are promotional. — [tik.tools](https://tik.tools/); [tik.tools WebSocket docs](https://tik.tools/websocket); [tik.tools vs Euler Stream](https://tik.tools/compare/eulerstream); [tiktool/tiktok-live-api on GitHub](https://github.com/tiktool/tiktok-live-api)
- BetterTok advertises a free developer WebSocket with 15 normalized event types. TikXander runs a local agent that streams gift, like, chat and follow events over a localhost WebSocket. — [BetterTok developer API](https://bettertok.app/developer-api); [TikXander game integration guide](https://www.tikxander.com/en/guides/integrate-your-games-with-tikxander)

### Inferences
- The release cadence shows these libraries break and get patched often: TikTok-Live-Connector had 4 patch releases in July–August 2026, and both libraries shipped the same day on 2026-07-22 and 2026-08-18. That points to protocol or schema changes on TikTok's side. Expect occasional outages of hours to days and plan to update dependencies before each stream.
- The Euler Stream free tier (25 WebSockets, 2,500 requests/day) is enough for one creator listening to their own stream. Paying ($50/month) mainly buys higher limits and priority, not a different legal status.
- For streaks: act only on `repeatEnd == true` (or `not event.streaking`) for streakable gifts, and act immediately for non-streakable gifts (`giftType != 1`). Value = gift diamond cost × `repeatCount`. Optionally show a "combo in progress" visual from the intermediate events.
- Read-only listening needs no TikTok login in either library. Login (session cookie or OAuth) is only needed for sending chat or the mobile flow, and that is where most account risk sits (see section 6).

### Gaps
- The libraries did not publish numeric latency. I found no measured end-to-end latency figures (gift sent → event received). WebSocket push is generally near-real-time, but this should be tested.
- The exact free-tier per-minute or per-connection limits for anonymous use (no API key) are not documented in what I could read; the eulerstream.com pricing page was not reachable.
- I could not verify the 2026 maintenance status of the Java (TikTokLiveJava) and Go (gotiktoklive) ports; the GitHub API was blocked for those repos in this session.
- I could not read BREAKING.md or GitHub issues for dated breakage incidents.

---

## 3. Ready-made creator tools: TikFinity, Streamer.bot, Social Stream Ninja, alternatives, and which are free or open source

### Takeaway
TikFinity is the de facto hub. It is free with a paid Pro tier, closed source, made under the "zerody" brand linked to the original TikTok-Live-Connector author, and provides alerts, TTS, overlays, actions & events, a desktop Event API/WebSocket and a Streamer.bot connector. Streamer.bot has no native TikTok support and relies on TikFinity. Social Stream Ninja is the main GPL-3.0 open-source option with WebSocket/API outputs. Many commercial "TikTok LIVE game" platforms exist; none are official TikTok products.

### Cited Findings
- TikFinity's site is titled "TikFinity: Free Alerts, TTS, and Overlays for TikTok LIVE". The app is also hosted at tikfinity.zerody.one, the same "zerody" domain as TikTok-Live-Connector's original author. — [tikfinity.com](https://tikfinity.com/); [tikfinity.zerody.one](https://tikfinity.zerody.one)
- TikFinity Pro, per search snippets of its site and setup guide: higher limits on actions, Sound Alerts, TTS and AI-voice credits; premium overlays; early access; more gift counters; up to 10 stream profiles. The free tier caps actions and sound alerts. The desktop app adds a local connection plus a chatbot, the Event API and keyboard-triggered sounds. Pricing is shown "in the app". — [TikFinity setup guide (blog)](https://blog.tikfinity.com/tikfinity-setup/); [tikfinity.com](https://tikfinity.com/)
- The TikFinity Mobile App Store listing shows "TikFinity Pro Yearly $59.99" and a $9.99 subscription. That is a separate mobile subscription, and the desktop Pro price was not confirmed. — [App Store: TikFinity Mobile](https://apps.apple.com/app/id6578448936)
- TikFinity exposes a WebSocket endpoint that "receives all events from your TikTok livestream" for custom games and widgets. A third-party tool says the default port is 21213, which I could not confirm in TikFinity's docs. — [TikFinity developer API page](https://tikfinity.zerody.one/tiktok/dapi); port claim via [itch.io vfinity tutorial](https://soursama.itch.io/vfinity/devlog/1313820/tutorial)
- Streamer.bot integration: enable the Streamer.bot WebSocket Server (Servers/Clients tab), then in TikFinity go to Setup → Streamer.bot Connection. Streamer.bot actions can then be triggered from TikFinity events, with username and nickname placeholders. "Allow Streamer.bot to push messages to TikFinity" lets Streamer.bot send chat to TikTok. — [TikFinity: Streamer.bot Integration](https://tikfinity.zerody.one/streamerbot-integration)
- Streamer.bot has no native TikTok support, and the official FAQ defers to TikFinity (quoted in section 1). The community bridge SBfinity exposes TikFinity events as Streamer.bot custom triggers but "cannot execute TikFinity's internal actions". — [Streamer.bot FAQ](https://docs.streamer.bot/faq/platform-support); [SBfinity (Notion)](https://rexbordz.notion.site/SBfinity-241088f4f93e80028773ff595fa0d5fa)
- Social Stream Ninja (SSN, steveseguin/social_stream) is GPL-3.0 and lists TikTok among supported platforms. It "offers WebSocket connections and APIs for custom integrations" plus webhooks, OBS, Stream Deck and MIDI. Caveats from its README: "Platform changes can temporarily break integrations", "Browser-based capture may pause when a source page is closed, minimized, or suspended", and "Automated posting and bots may be restricted by the source platform". It has a dedicated TikFinity-setup doc and a TikTok guide. — [steveseguin/social_stream](https://github.com/steveseguin/social_stream); [SSN TikFinity setup](https://socialstream.ninja/docs/tikfinity-setup.html); [SSN TikTok guide](https://socialstream.ninja/docs/tiktok-guide.html)
- TikControl is a commercial TikFinity alternative for Windows. It offers TTS, sound alerts, actions & events (min-diamond triggers, repeat count, superfan and subscription alerts), overlays usable as a LIVE Studio "link source" or OBS browser source, battle analytics, game integrations (Minecraft, GTA, Resident Evil) and TikFinity settings import. The source is an affiliate-linked tutorial. — [TikTok @livetutorials video](https://www.tiktok.com/@livetutorials/video/7646514693946805526)
- Crowd Control lets TikTok streamers bind game effects to Gifts, Coins, Likes, Follows and Shares. — [Crowd Control for TikTok](https://crowdcontrol.live/tiktok)
- Open-source interactive-game projects: vamnguyen/tiktok-live-games ("open-source platform… viewers can play games by typing commands in chat", with a TikTok Bridge SDK for HTML5 games, not affiliated with TikTok) and luccs0x0x/interatiok (an interactive game engine reacting to donations, likes and gifts). — [vamnguyen/tiktok-live-games](https://github.com/vamnguyen/tiktok-live-games); [luccs0x0x/interatiok](https://github.com/luccs0x0x/interatiok)
- TikTok LIVE Studio's built-in features: early coverage (2021) noted "No browser sources, or alerts". Newer third-party guides list alerts, goals and LIVE widgets, plus "link sources" for third-party overlays. — [TechCrunch 2021](https://techcrunch.com/2021/12/15/tiktok-is-testing-a-desktop-streaming-software-called-tiktok-live-studio/); [TikTok @livetutorials video](https://www.tiktok.com/@livetutorials/video/7646514693946805526)

### Inferences
- Free and open source: TikTok-Live-Connector (AGPL), TikTokLive (modified AGPL), TikTokLiveSharp, Social Stream Ninja (GPL-3.0), obs-websocket (GPL-2.0), and the small game projects above. Free but closed: TikFinity's free tier. Paid: TikFinity Pro, TikControl, Crowd Control (pricing not checked), Euler Stream Business, tik.tools paid tiers, and the hosted game platforms.
- The fastest no-code path is TikFinity desktop → its WebSocket/Event API or Streamer.bot → game or OBS. The most controllable free path is an own Node or Python bridge built on TikTok-Live-Connector or TikTokLive.
- TikFinity and TikTok-Live-Connector probably share lineage (the zerody brand) and therefore the same underlying reverse-engineered access. Switching between them does not change the ToS posture.

### Gaps
- I found no evidence of an official TikTok partnership with TikFinity, and no official TikTok statement about it. TikFinity's site could not be fetched to check its own wording. A Gigwise article ("Meet the Tool Many TikTok LIVE Streamers Rely On") exists but was not read. — [Gigwise](https://www.gigwise.com/?p=66245)
- TikFinity's desktop Pro monthly or yearly price and its exact WebSocket JSON schema and port were not confirmed from primary docs.
- StreamElements and Streamlabs TikTok event support (alerts from TikTok gifts) was not confirmed. Streamlabs does document TikTok streaming access, which is covered in section 5.
- TikFinity is not open source as far as I could find; no public repo turned up.

---

## 4. How events reach an engine (UE5, Unity, Godot) or a browser overlay in OBS: protocols, middleware, queueing and dedup

### Takeaway
The robust pattern is a local hub. A Node or Python process (or TikFinity's local WebSocket) receives TikTok events, normalizes them to JSON, queues and deduplicates them, then rebroadcasts over a local WebSocket (127.0.0.1) to the game and to OBS browser-source overlays. obs-websocket v5, built into OBS 28+, handles scene and source control. UE5 has an engine WebSockets module with several Blueprint wrapper plugins, Unity has a dedicated C# port, and Godot has an obs-websocket client and general WebSocket support.

### Cited Findings
- obs-websocket is "included by default with OBS Studio 28.0.0 and above". It is a 5.x server on default port 4455 with an auto-generated password (changeable under Tools → obs-websocket Settings), licensed GPL-2.0. Listed client libraries include obs-websocket-js (JavaScript), simpleobsws and obsws-python (Python), obs-websocket-gd (Godot 4), goobs (Go) and obs-websocket-java. Listed use cases include changing overlays based on the scene and automating scene switching with third-party tools. — [obsproject/obs-websocket](https://github.com/obsproject/obs-websocket)
- UE5: BlueprintWebSocket is "built around the engine's WebSockets module" with no other dependencies and supports multiple simultaneous sockets. — [BlueprintWebSocket docs](https://pandoa.github.io/BlueprintWebSocket/unreal-engine-5-websocket-plugin)
- SimpleWebSocket (UE) supports ws:// only and warns: "Do not use `localhost`… Always use `127.0.0.1` to avoid connection issues in packaged games." — [Unreal forums: SimpleWebSocket](https://forums.unrealengine.com/t/mengzhishanghun-simplewebsocket/2573828)
- The Epic Marketplace "WebSocket Server" plugin lets Unreal host the server, with Blueprint events for connect, disconnect and message. — [Unreal Marketplace: web socket server](https://docs.unrealengine.com/marketplace/en-US/product/web-socket-server)
- ZLive Bridge V1.0 (Fab) is a commercial UE plugin that exposes TikTok Chat, Like, Gift and Member as Blueprint events, with a UMG overlay and a demo mode, for UE 5.4–5.6 Win64. It notes that "optional local connectors (distributed separately)" are used, so it is not a TikTok reader by itself. — [Unreal forums: ZLive Bridge](https://forums.unrealengine.com/t/dztfix-katsu-zlive-bridge-v1-0/2656669); [Fab listing](https://www.fab.com/listings/fb6473aa-8224-41ae-85fb-60a3fc68277c)
- Unity: TikTokLiveSharp ships TikTokLiveUnity (see section 2). — [frankvHoof93/TikTokLiveSharp](https://github.com/frankvHoof93/TikTokLiveSharp)
- Managed-service message shape example: tik.tools sends JSON objects with `event` and `data` fields over `wss://`, and its guide shows Unity, C++ and web games consuming them. — [tik.tools WebSocket API](https://tik.tools/websocket); [tik.tools game guide](https://tik.tools/guides/tiktok-live-game)
- TikXander's design keeps TikTok credentials out of the game client by running a local agent that streams events on localhost. — [TikXander guide](https://www.tikxander.com/en/guides/integrate-your-games-with-tikxander)
- Social Stream Ninja offers WebSocket, APIs and webhooks for custom integrations, and can act as a multi-platform aggregator in front of a game. — [steveseguin/social_stream](https://github.com/steveseguin/social_stream)

### Inferences
These are design recommendations built from the cited facts, not sourced claims.
- **Reference architecture (Windows 11):**
  1. Node.js `tiktok-live-connector` (or Python `TikTokLive`) in read-only mode, no session cookie, with a free Euler API key.
  2. A normalizer that maps events to `{type, user, giftId, giftName, diamonds, count, ts, msgId}`.
  3. A queue with priority by diamond value and rate-limiting so expensive gifts are not starved by like spam.
  4. A local WebSocket server on `ws://127.0.0.1:<port>`, consumed by UE5 (BlueprintWebSocket or the engine WebSockets module), Unity (C#), Godot, and OBS browser-source overlays (plain JS `WebSocket`).
  5. An optional obs-websocket client in the same middleware to switch scenes or toggle sources on big gifts.
  - Alternative no-code front end: replace steps 1–2 with TikFinity's desktop WebSocket or Streamer.bot.
- **Streaks:** fire the "big" game effect on `repeatEnd`. Optionally fire small per-tick effects from the intermediate events, and use a fallback timeout (a few seconds) in case the end event is lost.
- **Deduplication:** key on the message ID or a tuple of (user ID, gift ID, group/combo ID, repeatCount). Ignore `processInitialData` history on reconnect, or set it to false, so old chat does not replay as new events after a reconnect.
- **Likes** arrive in batches, so aggregate them per time window instead of spawning one game event per like.
- **Separate processes:** running the bridge as its own process isolates TikTok protocol breakages from the game build. Only the bridge needs updating when the library ships a fix.
- **OSC:** OSC is an option for UE5 (OSC plugin) or TouchDesigner, but I found no TikTok-specific OSC tooling. WebSocket is the common denominator.

### Gaps
- I found no authoritative Godot-specific TikTok bridge documentation, and did not verify Godot's built-in WebSocketPeer docs in this session.
- I found no benchmark of bridge latency or event-drop rates.

---

## 5. Getting video to TikTok: LIVE Studio vs OBS with an RTMP stream key, eligibility, encoder settings, capture options and phone alternative

### Takeaway
Going LIVE generally needs roughly 1,000 followers and an age of 18+, and both vary by region. Having LIVE access does not guarantee an RTMP stream key. Keys come from TikTok LIVE Center/Producer or LIVE Studio and usually change per session. OBS can stream with a key: use H.264 CBR, a 1080x1920 canvas and roughly 2.5–4 Mbps per third-party guides. HEVC and AV1 support on TikTok ingest is unconfirmed. Without a key, TikTok LIVE Studio can capture the game window directly or take OBS via Virtual Camera.

### Cited Findings
- Eligibility: several guides cite 18+ and 1,000 followers, possibly varying by region and account age. One guide says 16+ (18+ in some regions). LIVE access is separate from RTMP and stream-key access: "your account has LIVE access but not RTMP access" can leave the producer page without a key. One guide says there is no legitimate way around the 1,000-follower rule. An agency's own marketing claims agency membership helps with key access. — [Restream: TikTok stream key](https://restream.io/learn/platforms/how-to-find-tiktok-stream-key/); [BIGVU 2026 guide](https://bigvu.tv/blog/how-to-get-a-tiktok-stream-key/); [OneStream 2025 guide](https://onestream.live/blog/tiktok-stream-key-guide/); [Rise Creator Network (agency)](https://www.risecreatornetwork.com/articles/how-to-get-tiktok-stream-key)
- Key mechanics: keys are generated in TikTok's LIVE Center or LIVE Studio, TikTok "does not give you a permanent stream key" (new key per session), and OBS uses Settings → Stream → Custom with a server URL commonly cited as an `rtmp://push.live.tiktok.com` address. — [OneStream: Stream on TikTok with OBS 2026](https://onestream.live/blog/how-to-stream-on-tiktok-with-obs/); [obs-versions.com guide](https://obs-versions.com/blog/obs-tiktok-live-streaming-guide)
- Encoder and bitrate:
  - Third-party guides recommend a 1080x1920 canvas. One suggests a 720x1280 output.
  - Bitrate is around 2,500–4,000 kbps ("TikTok does not benefit from higher bitrates like Twitch does").
  - Hardware NVENC is preferred, with x264 as fallback.
  - Use H.264 for RTMP unless HEVC/AV1 support is confirmed, since those need Enhanced RTMP on the ingest side (Dacast general guidance).
  - I found no official TikTok statement on HEVC or AV1. — [EveZone 2026 TikTok OBS settings](https://evezone.evetech.co.za/game-desk/best-obs-settings-for-tiktok-live-streaming-in-sa-2026); [Dacast best OBS settings 2026](https://www.dacast.com/blog/best-obs-studio-settings-2/); [OBSBOT guide](https://www.obsbot.com/blog/live-streaming/how-to-stream-on-tiktok-with-obs)
- TikTok LIVE Studio supports Game Capture, Window Capture and Display Capture for PC games. To route OBS into LIVE Studio, a BytePlus guide uses OBS Virtual Camera selected as the camera source in LIVE Studio; this is a third-party workaround, not a documented native feature. LIVE Studio is not available to every account in every region. — [OBSBOT: stream games on TikTok](https://www.obsbot.com/blog/live-streaming/how-to-stream-games-on-tiktok); [Insta360 guide](https://www.insta360.com/blog/tips/how-to-stream-games-on-tiktok.html); [BytePlus: Going live on TikTok via OBS](https://docs.byteplus.com/docs/byteplus-livesaas/Going_live_on_TikTok_via_Digital_Human_and_OBS); [Grokipedia: TikTok Live Studio](https://grokipedia.com/page/TikTok_Live_Studio) (low-quality source)
- Third-party overlays such as TikFinity and TikControl can be added in LIVE Studio as a "link source", or in OBS as a browser source at 1080x1920. — [TikTok @livetutorials video](https://www.tiktok.com/@livetutorials/video/7646514693946805526)
- Streamlabs has a support article titled "TikTok LIVE Streaming Access Revoked (No Violations)", which indicates that third-party streaming access can be withdrawn. I could only see the title. — [Streamlabs support](https://support.streamlabs.com/hc/en-us/articles/35259691237147-TikTok-Live-Streaming-Access-Revoked-No-Violations); [Streamlabs: how to go live on TikTok](https://streamlabs.com/content-hub/post/how-to-go-live-on-tiktok)

### Inferences
- For a new faceless account without a key, the realistic PC path is TikTok LIVE Studio capturing the game window, plus overlays added as link sources or OBS fed through Virtual Camera. Once an RTMP key is granted, OBS-direct gives more control.
- Safe default: H.264 (NVENC), CBR, 30 or 60 fps, keyframe interval of 2 s, 1080x1920 canvas, and 2.5–4 Mbps (or up to about 6 Mbps if stable), then test. Treat AV1/HEVC as unsupported until TikTok confirms it.

### Gaps
- No official TikTok help-center page on ingest specs (codec, maximum bitrate, keyframe interval) was reachable. All encoder numbers come from third-party guides.
- I did not research an Android-phone route (screen-share LIVE from a phone, or a phone as camera) in depth.
- I could not confirm current eligibility thresholds for Turkey specifically.

---

## 6. Risks: ToS clauses, reported bans, session-cookie exposure, TikTok's stance on TikFinity, data privacy

### Takeaway
Every event-reading route relies on reverse-engineered access. TikTok's ToS prohibits automated scraping and data extraction without written approval, and prohibits reverse engineering, so these routes are a ToS violation in principle even when read-only. However, I found no documented, verifiable cases of accounts banned purely for read-only listening. The concrete, documented risks are:
1. automated posting or chat spam with a session cookie (explicitly warned to risk suspension);
2. handing a session cookie to a third-party sign server;
3. LIVE restrictions triggered by on-screen or TTS content from viewers (the creator is responsible);
4. unofficial reports that TikFinity follower alerts get streams limited;
5. outages when TikTok changes its protocol.

### Cited Findings
- ToS on scraping: TikTok's Terms prohibit extracting platform data by bots or automated software "except as approved in writing", per a third-party clause tracker. — [ConductAtlas: prohibition on automated scraping](https://conductatlas.com/platform/tiktok/tiktok-terms-of-service/provision/CA-P-018595/prohibition-on-automated-scraping-of-platform-data/); [ConductAtlas: scraping and data extraction](https://conductatlas.com/platform/tiktok/tiktok-terms-of-service/prohibition-on-scraping-and-data-extraction/)
- ToS on reverse engineering: TikTok's Terms bar reverse engineering, decompiling or disassembling the platform, including its algorithms, code or infrastructure, without written authorization. The current US version references the TikTok USDS Joint Venture. A 2024 archived copy also lists reverse engineering among prohibited acts. — [ConductAtlas: prohibition on reverse engineering](https://conductatlas.com/platform/tiktok/tiktok-terms-of-service/provision/CA-P-018586/prohibition-on-reverse-engineering-platform/); [GMU law course PDF (2024 snapshot)](https://www.law.gmu.edu/assets/files/academics/schedule/2024/fall/law102_r04_first_reading_part_two.pdf)
- Both main libraries call themselves reverse-engineering projects that are unofficial and unaffiliated with ByteDance. — [TikTok-Live-Connector](https://github.com/zerodytrash/TikTok-Live-Connector); [TikTokLive](https://github.com/isaackogan/TikTokLive)
- Session-cookie risk, quoted from the README: "sessionid is your account's session token (keep it secret)". `authenticateWs` is "Disabled by default since signing is done by a third-party service; enabling it sends your session credentials to that service." `sendMessage` and `useMobile` require the session. — [npm README tiktok-live-connector 2.5.0](https://registry.npmjs.org/tiktok-live-connector)
- Older TikTok-Live-Connector README: the sessionId "is used to send automated chat… spamming messages can lead to the suspension of your TikTok account". — [unpkg README (older fork/version)](https://unpkg.com/@qixils/tiktok-live-connector@1.1.6/README.md)
- Read-only mode needs no credentials: "No login, no credentials or app are required." — [PyPI TikTokLive](https://pypi.org/project/TikTokLive/)
- TikFinity's own blog says TikTok's Community Guidelines hold creators responsible for third-party tools they enable, including tools that read comments aloud or show them on screen. A LIVE restriction is the first step, and repeat violations or evasion can lead to bans. — [TikFinity blog: TikTok LIVE rules](https://blog.tikfinity.com/tiktok-live-rules/)
- A LIVE-creator newsletter claims TikTok often limits streams that use TikFinity follower alerts, even though LIVE Studio offers a similar built-in feature. This is anecdotal. — [Live Success newsletter #3](https://live-success.beehiiv.com/p/dear-tiktok-live-success-3)
- A YouTube video is titled "TIKFINITY CAN GET YOUR TIKTOK BANNED!", but its description only says use "can lead to" bans or restricted LIVE access. That is a claim, not evidence. — [YouTube](https://www.youtube.com/watch?v=8JctbuLGd6s)
- Streamlabs documents cases of TikTok LIVE streaming access being revoked with "No Violations" (title only seen). — [Streamlabs support](https://support.streamlabs.com/hc/en-us/articles/35259691237147-TikTok-Live-Streaming-Access-Revoked-No-Violations)
- Euler Stream and tik.tools both disclaim any affiliation with TikTok. — [euler-api-sdk](https://github.com/eulerstream/euler-api-sdk); [tik.tools](https://tik.tools/)
- SSN warns that "automated posting and bots may be restricted by the source platform". — [steveseguin/social_stream](https://github.com/steveseguin/social_stream)

### Inferences
- **Risk ranking** (my assessment, not sourced as such):
  - Lowest: read-only listening with no login, via the libraries or TikFinity. It is still a ToS grey or black area but carries no reported enforcement.
  - Medium: showing or reading viewer text on stream (TTS or on-screen chat), because of moderation responsibility. Mitigate with filters, gifters-only TTS and banned-word lists.
  - High: automated chat posting with a session cookie, especially through a third-party sign server; this means credential exposure plus documented suspension risk.
  - Platform risk: TikTok could revoke LIVE or RTMP access without stated cause, as the Streamlabs article suggests.
- **Operational risk:** the whole game depends on a single unofficial chain (TikTok protocol → Euler sign server → library). Build a fallback, such as a "manual/test mode" or a TikFinity backup, and do not promise viewers features that break when that chain fails.
- **Data privacy:** viewer usernames, nicknames, avatars and comments are personal data. A creator in Turkey storing or logging them would fall under KVKK (and GDPR for EU viewers), so minimize logging and avoid persistent databases of viewer identities. No source was found specifically on this; it is a legal inference to verify.
- **No partnership:** I found no evidence that TikTok endorses or partners with TikFinity. Its wide use suggests tolerance, but that is not permission.

### Gaps
- I found no Reddit or GitHub-issue reports with verifiable details (dates, account state, tool used) of bans caused by read-only connectors. Absence of evidence is not evidence of safety.
- I could not read the official tiktok.com/legal Terms of Service text directly; the clauses above come from a third-party tracker and a 2024 snapshot.
- I could not determine whether TikTok actively detects webcast clients signed through Euler Stream.

---

## 7. Free or local TTS for reading comments in OBS (brief)

### Takeaway
The creator tools bundle TTS: TikFinity (free with limits, more TTS and AI-voice credits in Pro) and TikControl (configurable prefixes, voices, members- or gifters-only). TTS output must go to the same audio device that LIVE Studio or OBS captures. Restricting TTS to gifters or members is the common safety practice.

### Cited Findings
- TikFinity provides TTS, and Pro raises TTS and "AI voice credits" limits. — [tikfinity.com](https://tikfinity.com/); [TikFinity setup guide](https://blog.tikfinity.com/tikfinity-setup/)
- TikControl TTS supports prefixes, username reading, voice selection and audio-output selection. The tutorial advises against letting everyone use TTS; members, gifters or superfans are safer. Output must match the device used by LIVE Studio or OBS. — [TikTok @livetutorials video](https://www.tiktok.com/@livetutorials/video/7646514693946805526)
- Creators are responsible for third-party tools that read comments aloud. — [TikFinity blog: TikTok LIVE rules](https://blog.tikfinity.com/tiktok-live-rules/)

### Inferences
- For a fully local, free pipeline, the custom bridge can pipe filtered comments into a local TTS engine and play the audio into OBS. Engine selection is covered by the AI-stack researcher.

### Gaps
- I did not identify which TTS engines TikFinity uses internally (cloud vs local).
