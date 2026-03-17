# Erlang/OTP Compatibility Report

> Generated: March 2026 (updated: March 9, 2026)
>
> Scope: `carbonio-erlang`, `carbonio-message-broker` (RabbitMQ), `carbonio-message-dispatcher` (MongooseIM)

## Current Versions

All versions are taken from `pkgver` in the respective PKGBUILD files.

| Repository                    | Package                       | `pkgver` | Upstream Software   |
| ----------------------------- | ----------------------------- | -------- | ------------------- |
| `carbonio-erlang`             | `carbonio-erlang`             | 26.2.5.2 | Erlang/OTP 26.2.5.2 |
| `carbonio-erlang`             | `carbonio-elixir`             | 1.17.2   | Elixir 1.17.2       |
| `carbonio-message-broker`     | `carbonio-message-broker`     | 0.2.4    | RabbitMQ 3.13.6     |
| `carbonio-message-dispatcher` | `carbonio-message-dispatcher` | 0.16.0   | MongooseIM 6.4.0    |

All three downstream packages depend on `carbonio-erlang` at both build and runtime
with **no version constraint**. They consume whatever OTP version `carbonio-erlang` provides.

## Upstream Erlang/OTP Compatibility Matrix

| Erlang/OTP  | RabbitMQ 3.13.x     | RabbitMQ 4.1.x  | RabbitMQ 4.2.x  | MongooseIM 6.4.0 | MongooseIM 6.5.0 |
| ----------- | ------------------- | --------------- | --------------- | ---------------- | ---------------- |
| 26.0 - 26.1 | Supported           | -               | -               | Supported        | -                |
| **26.2.x**  | **Supported (max)** | Supported (min) | Supported (min) | Supported        | **Dropped**      |
| **27.x**    | **Not supported**   | Supported (max) | Supported (max) | Supported        | Supported (min)¹ |
| 28.x        | Not supported       | Not supported   | Partial         | Not supported    | Supported (max)  |

¹ MongooseIM 6.5.0's `rebar.config` still says `{require_min_otp_vsn, "26"}` (upstream
oversight — not bumped when OTP 26 was dropped). The **actual** minimum is OTP 27: release
notes explicitly state "Drop support for Erlang 26", and CI only tests OTP 27.3.4.2 and 28.0.2.

Sources:

- RabbitMQ: <https://www.rabbitmq.com/docs/which-erlang>
- MongooseIM 6.4.0: `{require_min_otp_vsn, "26"}` in `rebar.config`
- MongooseIM 6.5.0: release notes say "Drop support for Erlang 26" and "Support Erlang 28".
  Note: the `rebar.config` in 6.5.0 still says `{require_min_otp_vsn, "26"}` — this is an
  upstream oversight (PR #4549 forgot to bump it). The actual supported range is OTP 27-28,
  as confirmed by CI which only tests 27.3.4.2 and 28.0.2.

## Where Support Starts and Drops

| Event                                 | OTP Version | Impact                                        |
| ------------------------------------- | ----------- | --------------------------------------------- |
| RabbitMQ 3.13.x maximum supported OTP | 26.2.x      | Cannot use OTP 27 without upgrading RabbitMQ  |
| RabbitMQ 4.0+ minimum OTP             | 26.2        | RabbitMQ 4.x still accepts OTP 26.2           |
| RabbitMQ OTP 27 support starts        | 27.x        | Requires RabbitMQ >= 4.0.4                    |
| MongooseIM OTP 27 support starts      | 27.x        | Added in MongooseIM 6.3.0, inherited by 6.4.0 |
| MongooseIM 6.4.0 minimum OTP          | 26          | `{require_min_otp_vsn, "26"}`                 |
| MongooseIM 6.5.0 **drops OTP 26**     | 27 (min)    | Cannot use OTP 26 with MongooseIM >= 6.5.0    |

## Current Status

The current combination (OTP 26.2.5.2 + RabbitMQ 3.13.6 + MongooseIM 6.4.0) is **valid**
but boxed in:

- **RabbitMQ 3.13.6** caps Erlang at OTP 26.2.x. Upgrading Erlang to 27 without
  upgrading RabbitMQ first **will break RabbitMQ** (it will fail to start).
- **MongooseIM 6.5.0** (latest upstream, Dec 2025) requires OTP 27 minimum.
  It cannot be adopted without upgrading both Erlang and RabbitMQ.
- **RabbitMQ 3.13.x** is the last pre-4.0 series and will stop receiving patches.

## Upgrade Ordering Constraint

Since all packages consume `carbonio-erlang` without version pinning, upgrades must be
coordinated. The order matters:

```text
1. Upgrade RabbitMQ to 4.1.x       (still works on OTP 26.2)
2. Upgrade Erlang to OTP 27        (now works for both RabbitMQ 4.x and MongooseIM 6.4.0)
3. Upgrade MongooseIM to 6.5.0     (requires OTP 27, now available)
```

Steps 2 and 3 can ship together since MongooseIM 6.5.0 requires OTP 27 and MongooseIM 6.4.0
supports it. The only hard constraint is that step 1 must land before step 2.

Upgrading `carbonio-erlang` to OTP 27 **before** upgrading RabbitMQ past 3.13.x will break
the message broker. In practice, steps 1 and 2 should ship in the same release cycle.

The existing branch `CO-2722` (Erlang 27.3 + Elixir 1.19.0) **must not be merged and
released** until `carbonio-message-broker` is also upgraded past RabbitMQ 3.13.6.

## Recommended Upgrade Targets

| Package                       | Current          | Target                             | Rationale                                                                                           |
| ----------------------------- | ---------------- | ---------------------------------- | --------------------------------------------------------------------------------------------------- |
| `carbonio-erlang`             | OTP 26.2.5.2     | **OTP 27.3**                       | Only major version satisfying both RabbitMQ 4.x and MongooseIM 6.5.0. Branch `CO-2722` exists.      |
| `carbonio-elixir`             | 1.17.2           | **1.19.x**                         | Current release. 1.17 also supports OTP 27 if minimal change is preferred. Branch `CO-2722` exists. |
| `carbonio-message-broker`     | RabbitMQ 3.13.6  | **RabbitMQ 4.1.x** (latest: 4.1.8) | Stable maintained series. Fully supports OTP 26.2 and 27. More conservative than 4.2.x.             |
| `carbonio-message-dispatcher` | MongooseIM 6.4.0 | **MongooseIM 6.5.0**               | Latest release; supports OTP 27-28. No reason to stay on 6.4.0 when upgrading to OTP 27. |

### Why RabbitMQ 4.1.x over 4.2.x

Both are valid targets. 4.1.x is recommended as the more conservative choice:

- 4.2.x has partial Erlang 28 support with known Khepri-related issues.
- The 3.13 to 4.x jump is already significant (classic mirrored queues removed,
  AMQP 1.0 native support, Khepri as optional metadata store).
- 4.1.x is the current LTS-adjacent stable series receiving active patches.

4.2.x is reasonable if you want the latest features and are comfortable with
the newer release.

## Available New Upstream Versions

> Data fetched from GitHub releases on March 9, 2026.

### Erlang/OTP

| Series | Current      | Latest Available | Released   | Notes                                       |
| ------ | ------------ | ---------------- | ---------- | ------------------------------------------- |
| OTP 26 | **26.2.5.2** | **26.2.5.17**    | 2026-02-20 | Patch-only updates; still supported         |
| OTP 27 | -            | **27.3.4.8**     | 2026-02-20 | Upgrade target (branch `CO-2722` uses 27.3) |
| OTP 28 | -            | **28.4**         | 2026-03-04 | Too new; limited downstream support         |
| OTP 29 | -            | 29.0-rc1         | 2026-02-11 | Release candidate only                      |

**Immediate low-risk action**: bump `carbonio-erlang` from 26.2.5.2 to **26.2.5.17** (patch
update within the same series). This is safe for all current downstream packages and picks up
~15 patch releases of bug/security fixes with zero compatibility risk.

### Elixir

| Series | Current    | Latest Available | Released   | Notes                                               |
| ------ | ---------- | ---------------- | ---------- | --------------------------------------------------- |
| 1.17   | **1.17.2** | **1.17.3**       | 2024-09-18 | Bug fix release; safe patch bump                    |
| 1.18   | -          | **1.18.3**       | 2025-03-06 | Supports OTP 25-27. Type system additions.          |
| 1.19   | -          | **1.19.5**       | 2026-01-09 | Supports OTP 25-27. Major type system improvements. |
| 1.20   | -          | 1.20.0-rc.2      | 2026-03-04 | Release candidate; requires OTP 27+.                |

**Immediate low-risk action**: bump `carbonio-elixir` from 1.17.2 to **1.17.3** (same series
patch). For the OTP 27 upgrade, target **1.19.5** (stable, supports both OTP 26 and 27) —
branch `CO-2722` currently has 1.19.0 which should be updated to 1.19.5.

### RabbitMQ

| Series | Current    | Latest Available | Released   | Notes                                                   |
| ------ | ---------- | ---------------- | ---------- | ------------------------------------------------------- |
| 3.13.x | **3.13.6** | **3.13.7**       | 2024-08-26 | Last in series; community-only support. OTP 26.2.x max. |
| 4.0.x  | -          | **4.0.9**        | 2025-04-14 | First 4.x series. Already superseded by 4.1.x.          |
| 4.1.x  | -          | **4.1.8**        | 2026-01-22 | **Recommended upgrade target.** OTP 26.2 - 27.x.        |
| 4.2.x  | -          | **4.2.4**        | 2026-02-17 | Latest series. Partial OTP 28 support.                  |

**No low-risk patch action**: 3.13.7 exists but community support for 3.13.x ended June 2024.
The upgrade path is to **4.1.8** as outlined in the Recommended Upgrade Targets section.

### MongooseIM

| Series | Current   | Latest Available | Released   | Notes                                                |
| ------ | --------- | ---------------- | ---------- | ---------------------------------------------------- |
| 6.3.x  | -         | **6.3.3**        | 2025-04-10 | Patch on previous series. OTP 26-27.                 |
| 6.4.x  | **6.4.0** | **6.4.0**        | 2025-06-25 | Already at latest. OTP 26-27.                        |
| 6.5.x  | -         | **6.5.0**        | 2025-12-08 | Drops OTP 26; requires OTP 27+ (CI tests 27.3 and 28.0 only). Adds OTP 28 support. `rebar.config` still says min OTP 26 (upstream bug). |

**Upgrade to 6.5.0** alongside the OTP 27 upgrade. 6.4.0 is a dead-end (no 6.4.1 exists), and
since the plan already commits to OTP 27, there's no reason to stay on 6.4.0. Ship MongooseIM
6.5.0 + OTP 27 together in step 2/3 of the upgrade ordering.

### Summary of Safe Immediate Patches

These can be done **right now** with no coordination required:

| Package           | From          | To            | Risk                     |
| ----------------- | ------------- | ------------- | ------------------------ |
| `carbonio-erlang` | OTP 26.2.5.2  | OTP 26.2.5.17 | None (same series patch) |
| `carbonio-elixir` | Elixir 1.17.2 | Elixir 1.17.3 | None (same series patch) |

### RabbitMQ 3.13 to 4.x Breaking Changes to Watch

- Classic mirrored queues are fully removed. Quorum queues are the replacement.
- Khepri is available as an optional metadata store (replacing Mnesia).
- Default exchange type and queue behaviors may differ.
- Consult the [4.0.0 release notes](https://github.com/rabbitmq/rabbitmq-server/releases/tag/v4.0.0)
  and [4.1.0 release notes](https://github.com/rabbitmq/rabbitmq-server/releases/tag/v4.1.0)
  for the full list.
