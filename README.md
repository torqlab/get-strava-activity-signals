# @torqlab/get-strava-activity-signals

Extract prompt-ready **semantic signals** from a Strava activity.

This package takes a Strava activity payload (as returned by the Strava API) and produces a small, typed `StravaActivitySignals` object: intensity, elevation, time-of-day, tags, and lightweight context from user text — plus derived signals like mood, style, subject, terrain, environment, and atmosphere.

It’s designed for **downstream prompt generation** (e.g. activity artwork / summaries) where you want consistent, guardrailed labels rather than raw, messy activity data.

## Install

Published to GitHub Packages.

```bash
npm i @torqlab/get-strava-activity-signals
```

Or with Bun:

```bash
bun add @torqlab/get-strava-activity-signals
```

If you haven’t used GitHub Packages before, you may need an `.npmrc` with the registry mapping:

```ini
@torqlab:registry=https://npm.pkg.github.com
```

## Quick start

```ts
import getStravaActivitySignals from '@torqlab/get-strava-activity-signals';
import type { StravaActivitySignals } from '@torqlab/get-strava-activity-signals';

const stravaActivity = {
  id: 123,
  type: 'Run',
  sport_type: 'Run',
  name: 'Morning Run',
  description: 'Easy miles in the park',
  distance: 5000,
  moving_time: 1500,
  total_elevation_gain: 60,
  start_date_local: '2024-01-01T07:10:00Z',
  gear: { name: 'Nike Pegasus' },
  commute: false,
};

const signals: StravaActivitySignals = getStravaActivitySignals(
  stravaActivity,
  // Your moderation/guardrails callback for any user-provided text.
  // If you don’t need it, pass `() => false`.
  (input) => input.toLowerCase().includes('forbidden'),
);

console.log(signals.core.intensity);
console.log(signals.derived.atmosphere);
```

## API

### `getStravaActivitySignals(activity, checkForbiddenContent)`

**Parameters**

- `activity`: a Strava activity object (shape aligned with Strava’s `GET /activities/{id}` response).
- `checkForbiddenContent`: `(input: string) => boolean` used to detect forbidden/unsafe content in any user-provided strings.
  - Used when extracting `semanticContext` from `name`/`description` and when extracting `brands` from `gear`.

**Returns**

- `StravaActivitySignals`

**Throws**

- If activity validation fails (e.g. missing required fields like `type` / `sport_type`, or invalid moving-activity value constraints).
- If signals validation fails and cannot be sanitized.

## Output: `StravaActivitySignals`

The result is split into:

- `core`: directly extracted/classified from the activity
- `derived`: higher-level labels derived from `core` signals

```ts
export interface StravaActivitySignals {
  core: {
    activityType: string;
    intensity: 'low' | 'medium' | 'high';
    elevation: 'flat' | 'rolling' | 'mountainous';
    timeOfDay: 'morning' | 'day' | 'evening' | 'night';
    tags?: string[];
    brands?: string[];
    semanticContext?: string[];
  };
  derived: {
    mood: 'calm' | 'intense' | 'routine' | 'playful' | 'focused';
    style: 'cartoon' | 'minimal' | 'abstract' | 'illustrated';
    subject: 'runner' | 'cyclist' | 'trail runner' | 'walker' | 'hiker' | 'swimmer' | 'athlete';
    terrain: 'mountainous terrain' | 'rolling hills' | 'flat terrain';
    environment: 'indoor training space' | 'outdoor training space';
    atmosphere:
      | 'soft morning light'
      | 'bright daylight'
      | 'warm evening glow'
      | 'dark night atmosphere'
      | 'soft neutral light';
  };
}
```

### Notes on core fields

- `activityType`: sourced from `sport_type` (falls back to `type`, then `'Unknown'`).
- `semanticContext`: keyword-based signals extracted from `name` and `description` (after moderation via `checkForbiddenContent`).
- `tags`: normalized tags (currently includes `commute` and supports known tags like `recovery`, `race`, `easy`, etc.).

## How it works (high level)

1. **Validate** the input activity (required fields + guardrails for moving activities)
2. **Extract core signals** (activity type, intensity, elevation, time-of-day, tags, brands, semantic context)
3. **Derive higher-level signals** (mood/style/subject/terrain/environment/atmosphere)
4. **Validate/sanitize** signals (filters forbidden semantic context when possible)

## Modules

This repo is organized as small, focused extractors/classifiers:

- Extractors: `extract-time-of-day-signals`, `extract-tag-signals`, `extract-semantic-context`, `extract-text-signals`, `extract-brand-signals`
- Classifiers: `classify-intensity`, `classify-elevation`, `classify-mood`, `classify-style`, `classify-subject`, `classify-terrain`, `classify-environment`, `classify-atmosphere`
- Guardrails: `validate-activity`, `validate-signals`

## Development

This package has **no runtime dependencies**. Tooling uses Bun + TypeScript.

```bash
bun test
bun run lint
bun run format
bun run build
```

## License

MIT
