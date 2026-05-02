---
name: model-routing
description: Determines which Xiaomi MiMo model to use when spawning subagents for Beads tasks. Use when dispatching work, creating issues with model preferences, or deciding which model fits a task.
---

# Model Routing

## Quick Start

When spawning a subagent, run this decision flow:

1. Check issue for `execution_suggested_model` → use if set
2. Check issue tags for `simple` or `complex` → use that
3. If no tag, classify from title + description
4. Look up model in routing table

## Routing Table

### Text / Code

| Complexity | Model | Example |
|------------|-------|---------|
| simple | MiMo-V2-Pro | "Add null check to auth.ts" |
| complex | MiMo-V2.5-Pro | "Implement progressive recall engine" |

### Multimodal

| Complexity | Model | Example |
|------------|-------|---------|
| simple | MiMo-V2-Omni | "What's in this screenshot?" |
| complex | MiMo-V2.5 | "Analyze this demo video" |

### TTS

| Use Case | Model |
|----------|-------|
| Standard | MiMo-V2.5-TTS |
| Voice clone | MiMo-V2.5-TTS-VoiceClone |
| Voice design | MiMo-V2.5-TTS-VoiceDesign |

## Complexity Rules

**Simple** signals:
- Single file, "fix"/"update"/"rename"/"typo"
- No dependencies, narrow scope

**Complex** signals:
- Multi-file, "implement"/"design"/"architecture"
- Dependencies, long context, reasoning chains

**Default**: complex (safer to use flagship)

## Issue Override

Set model explicitly on any issue:
```bash
bd update <id> --set-metadata execution_suggested_model=MiMo-V2-Pro
```

Or tag complexity:
```bash
bd create "Title" -t task --add-tag simple
```

## Task Type Detection

| Keywords | Type |
|----------|------|
| tts, speech, voice | tts |
| image, video, audio, screenshot | multimodal |
| everything else | text |
