VOXACHAT VIP IMPLEMENTATION PACKAGE — REVIEW / INTEGRATION BUILD

HONEST STATUS
VIP Center HTML, authoritative wallet/RPCs, equipment persistence, owner/protection helper and scheduler SQL are supplied. No live database/browser tests have been performed. This is NOT a fully deployed room/payment system. Original crown/effect artworks are NOT included: image generation failed due to account quota. Missing assets display explicit missing-artwork messages, not fake crowns. No claim of production readiness.

INSTALL ORDER
1. Back up your site/database. Review 01-vip-schema.sql on staging, then run using Supabase SQL Editor as database owner.
2. This creates new voxa_vip_* tables, and does NOT migrate/alter old vip_profiles. Migrate only verified balances and dates using trusted backend/SQL, never browser writes. Old vip_profiles data must not be considered authoritative until its write policies have been audited.
3. Enable Cron and run 02-schedule.sql once. Hourly granularity means cards can appear up to an hour after scheduled boundary; signed-in refresh also calls an idempotent grant.
4. Replace your VIP Center with vip-center.html. Serve on same origin as existing login. The publishable Supabase key is already set; never add a service-role key to HTML.
5. Install real assets as described below, and connect room/chat/payment integrations. Test all failure paths before launch.

CUSTOMIZATION / ORIGINAL ARTWORK
Frontend CONFIG handles badges and preview privilege catalog. Backend voxa_vip_rules is authoritative for EXP and card quantities. Update both to match; privilege names/unlock list currently live in frontend only and are NOT enforcement for other app features.
Badge frontend paths: assets/vip/badge-1.webp through badge-10.webp. Supply detailed original/licensed metallic artwork at these paths (or edit URLs).
voxa_vip_items has 50 slots: badge, frame, bubble, entry and profile for each of VIP 1–10. Set asset_url to real image URLs, animation_url to actual video URLs (WebM/MP4). This build's animation player supports video files, not Lottie JSON. Animated image assets can be asset_url. Originals were not fabricated here. Set names/art and enabled appropriately.
Entry 1–5 Normal, 6–10 Royal, all separate slots. Server equip rejects missing assets. Badge selection is automatic based on actual VIP; user cannot equip a higher badge. Other item kinds can be equipped if eligible; renderers must fetch/validate the actual current entitlement on each feature use. Public profile decoration endpoint remains to be implemented with a privacy-safe projection.

CONFIRMED BUSINESS RULES
95-day active periods; weekly Kick/Global grants for VIP 7,8,9,10 = 3,5,7,10; one Room card daily. Each issued batch lasts 7 days. Kick protection starts at VIP 7. Protected targets cannot be kicked by admins/cards, only room owner. Room owner cannot be kicked. Voluntary leave allowed. Platform moderation separate.
Normal and Royal Entry privilege categories are listed once, with individual artwork slots for each level. Extra obsolete duplicate VIP10 Royal Entry has been removed; do not compare entry counts to old file's 25. Actual catalog has 26 entries, NOT a claimed 52.

IMPLEMENTATION CHOICES THAT STILL REQUIRE PRODUCT SIGN-OFF
Weekly/daily boundaries are anchored to period_start, not calendar midnight. First grant is current bucket at activation/sync; keep Cron running to grant offline. Grants do not backfill previously missed days/weeks. In-period upgrade does not duplicate/refill already granted current buckets; higher quota applies next bucket. A VIP 7 upgrade late into an existing low-VIP period uses current period anchor; decide whether premium activation needs a distinct grant anchor.
Cards retain their issued 7-day timestamp but redemption requires active VIP7+. Thus a last-week card cannot be used once VIP expires. Older unexpired batches can be redeemed during a new eligible period. These edge policies were NOT explicitly confirmed by user and must be reviewed.
A genuine 95-day period is stored by trusted server as period_start and period_end. The schema accepts dates rather than forcing 95 days; backend must write the agreed duration. Existing EXP thresholds retained, not newly confirmed. No recharge-to-EXP conversion invented. No automatic reset/renewal or payment credit RPC is supplied.

ROOM INTEGRATION CONTRACT (existing room schema not provided)
Your trusted server must know room owner, actor membership/role, target membership and bans. Never accept p_owner or p_user supplied unverified by client.
NORMAL ADMIN KICK: require ordinary room permissions AND voxa_vip_can_kick(actor,target,real_owner). If denied do not remove membership.
KICK CARD: in ONE database transaction, validate room membership/target, helper rule, ban/rejoin policy, then call voxa_vip_redeem(actor,'kick',requestUUID,canonicalContext), then remove target membership/write authoritative kick event. If redeem returns false, request was already processed: do NOT repeat the action. If removal fails the whole transaction rolls back so card is not lost. Calling separate Supabase RPCs is NOT one transaction; implement combined room RPC once schema is supplied.
ROOM/GLOBAL MESSAGE: authenticate, validate access/content/moderation/rate limits; in same DB transaction redeem appropriate card and INSERT durable message/outbox record. Broadcast after commit, with retry/outbox support. Neither the RPC nor HTML currently sends real messages.
ENTRY: trusted room join handler validates membership and actual level, resolves equipped eligible entry (or configured level default), and broadcasts canonical user/name/avatar/effect event to authorized room subscribers. Deduplicate event IDs and queue UI animations. Do not let arbitrary clients forge premium entries. Do not play effects merely for preview visits. Frame/bubble rendering likewise must be integrated.
voxa_vip_can_kick is a protection helper, NOT an authorization function; true does NOT mean an arbitrary actor has admin rights or a card. Those checks belong to combined room action.

GIF AVATAR
This package includes GIF Avatar eligibility in catalog but does not provide upload RPC/storage policies because existing bucket rules were not supplied. Reuse only a server-authorized upload endpoint that checks actual VIP >=6, MIME/signature, size and ownership. Do not treat UI gating as permission. Previous upload demo is intentionally not represented as a secured final feature.

SECURITY
New authoritative tables have RLS and no browser table writes/reads; approved RPC exposes only own wallet/state. Security-definer functions pin search_path and revoke public/anon execution. Internal functions are inaccessible to ordinary clients. Never grant users write access to account EXP, periods, cards or asset catalog. Server-only redeem must be combined transactionally with actual action; never expose as a standalone spend button. Existing legacy tables/policies still need separate audit. Generated SQL has not been parsed/executed against your schema here.

TRUSTED ACTIVATION EXAMPLE (backend/SQL only; not payment integration)
After validating real payment and deriving correct EXP, a trusted backend writes a voxa_vip_accounts row with auth user UUID, exp, period_start and period_end=period_start+interval '95 days'. Do not reset an active period on every recharge unless approved renewal rules require it. Use idempotent payment IDs and transactions in the real payment flow.

TEST CHECKLIST
Anonymous preview; no account; expired period; VIP0/6/7/10; 3/5/7/10 quotas; cron replay; concurrent redemption; duplicate request UUID; same UUID wrong context; daily batch expiry; in-period upgrade; last-day cards; protected target; owner target; failed kick transaction rollback; item eligibility/asset failure; logout data clearing; reduced-motion handling in final room player; original asset licensing; real payment verification.
