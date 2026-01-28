---
title: Dependency-Based Parallelization
impact: CRITICAL
impactDescription: 2-10x improvement
tags: async, parallelization, dependencies
---

## Dependency-Based Parallelization

When operations have partial dependencies, start independent work immediately and chain dependent work. This maximizes parallelism without waiting unnecessarily.

**Problem: profile waits for config unnecessarily**

```typescript
const [user, config] = await Promise.all([fetchUser(), fetchConfig()]);
const profile = await fetchProfile(user.id); // config already done, wasted time
```

Timeline: `[user + config] -> [profile]` (2 sequential steps)

**Solution: chain dependent operations, parallelize independent ones**

```typescript
const userPromise = fetchUser();
const profilePromise = userPromise.then((user) => fetchProfile(user.id));

const [user, config, profile] = await Promise.all([
  userPromise,
  fetchConfig(),
  profilePromise,
]);
```

Timeline: `[user + config]` and `[user -> profile]` run in parallel (profile starts as soon as user completes, doesn't wait for config)

**More complex example:**

```typescript
// user -> profile -> settings (chain)
// config (independent)
// permissions depends on user (parallel with profile)

const userPromise = fetchUser();
const profilePromise = userPromise.then((u) => fetchProfile(u.id));
const settingsPromise = profilePromise.then((p) => fetchSettings(p.id));
const permissionsPromise = userPromise.then((u) => fetchPermissions(u.id));

const [user, config, profile, settings, permissions] = await Promise.all([
  userPromise,
  fetchConfig(),
  profilePromise,
  settingsPromise,
  permissionsPromise,
]);
```

**Key insight:** Create all promises first, then `Promise.all()` at the end. Each promise starts executing immediately when created, and `.then()` chains execute as soon as their dependency resolves.
