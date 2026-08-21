# Format Time

| spec-id-prefix |
|----------------|
| PR-TIME-*      |

Format internal nanosecond time values to human-readable strings for axis ticks, cursor labels, tooltips, and the detail strip.

```ts
formatTime(ns, mode='time', opts?: { unit?, tickStepNs?, clockFreqMHz? }): string
formatTimeParts(ns, mode='time', opts?): { value: string; unit: string }
formatAxisTime(ns, mode='time', opts?): string
formatCursorTime(ns, mode='time', opts?): string
timeScaleUnitFromNsQuantum(quantumNs): TimeScaleUnit
resolveTimeUnitFromVisibleRange(spanNs): TimeScaleUnit
resolveClockFreqMHz(summary?): number | undefined
nsToCycles(ns, clockFreqMHz): number
```

## Behavior

**Internal representation.** All time values throughout the system use nanoseconds internally. Conversion to display units happens only at the formatting layer.

**Modes.** `time` = wall time with auto `TimeScaleUnit` (`s` / `ms` / `us` / `ns`). `cycles` = display CPU clocks via `cycles = ns × freqMHz / 1000` with `freqMHz` from `resolveClockFreqMHz` (`currentFreq ?? ratedFreq`, MHz); missing freq → `—`.

**Cycles mode (I-Q14).** Labels are a **time×freq conversion** for axis, cursor, tooltip, and detail — not per-event PMU `*_total_cycles` from `PipeUtilization`. Prefer **Current Freq** over Rated when both exist. On `data/out.rep`, `aiv_total_cycles / aiv_time(us)` matches OpBasicInfo `Current Freq` (1650), so the formula is consistent with measured block cycles for that fixture. Assumes swimlane ns timestamps share the AIC clock domain; do not use `HardwareInfo.ai_core_frequency_MHZ` for this path.

**Auto scale (time mode).** Viewport chrome uses `resolveTimeUnitFromVisibleRange(end − start)`. Overview / total axis uses major-tick step from span×width (`resolveTimeUnitFromAxisDensity` in axisRuler) — brush window must not change overview unit.

**Tooltip/detail formatting.** `formatTime` in time mode shows 3 decimal places (integer ns). `formatTimeParts` returns value and unit separately for the detail card (`7419` under `Start (ns)`); `formatTime` joins them. Cycles use a plain count with ` cycles` suffix.

**Axis tick formatting.** `formatAxisTime` adapts decimals from `tickStepNs`. Compact cycles suffix `cyc`. Origin → compact zero (`0ms` / `0cyc`).

**Cursor formatting.** Time mode: `MM:SS.mmm` in the resolved scale (sketch: 4.456ms → `00:04.456`). Cycles mode: plain cycle count (not clock-style).

## Acceptance Criteria

1. **PR-TIME-001** — time-mode format by scale unit.
1. **PR-TIME-002** — cursor MM:SS.mmm / cycles count.
1. **PR-TIME-002b** — visible-range / quantum resolvers.
1. **PR-TIME-003** — axis decimals follow tick step.
1. **PR-TIME-004** — compact axis zero.
1. **PR-TIME-005** — cycles conversion, freq resolve, and formatTimeParts.

## Edge Cases

Zero → compact `'0ms'` on axis (via PR-TIME-004); tooltip `formatTime(0)` still `'0.000 ms'`. NaN/Infinity → `'—'`. Negative cursor → clamped to 0. Cycles without freq → `'—'`.

## Dependencies

I-Q14 — Time (auto) vs CPU clocks; freq from `currentFreq ?? ratedFreq`; see [INTERIM_DECISIONS I-Q14](../../docs/context/INTERIM_DECISIONS.md#i-q14--cpu-clocks-from-opbasicinfo-freq).

## Changelog
- **2026-08-24** — Document cycles-mode freq source, formula, fixture cross-check, and caveats; formatTimeParts with mode/opts.
- **2026-08-21** — Time vs cycles modes; auto scale; seconds support.
- **2026-08-13** — PR-TIME-005 `formatTimeParts` for the detail card's unit-in-label layout.
- **2026-08-07** — PR-TIME-004 compact axis zero.
- **2026-08-05** — Initial spec. Core behaviors established.
