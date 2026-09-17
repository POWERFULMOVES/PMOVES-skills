---
name: pmoves-voice-fabric
description: "Use when wiring voice engines via flute-gateway or MCP."
---

# PMOVES Voice Fabric — flute-gateway + MCP + dsh agents

flute-gateway (:8055) is THE voice gateway: 8 providers behind a VoiceProvider ABC (synthesize/synthesize_stream/recognize) — vibevoice, voicebox, omnivoice (default), whisper (STT), ultimate_tts, kokoro, cloning pair. REST /v1/voice/{config,synthesize,synthesize/audio,recognize,personas}. CHIT voice-attribution events built in.

## Fleet voice services
flute-gateway 8055, cast-tts-gateway, kokoro-tts, vibevoice-realtime, voice-relay, voice-sampler, audio-reprocess, media-audio, ffmpeg-whisper. Pinokio side: pmoves-services launcher + Ultimate TTS Studio (feeds flute via ULTIMATE_TTS_URL).

## Plan (docs/services/VOICE_FABRIC_PLAN.md)
1. **Layer 1** — flute `/mcp` mount (FastMCP on the FastAPI app) exposing voice_synthesize/stream/recognize/personas/engines_list/capabilities → mcp_inventory entry `pmoves-flute-voice` (http, local, clients [hermes] after live verify)
2. **Layer 2** — `pmoves/config/voice/engines/*.yaml` capability manifests per engine: latency class, cost class, presets, strengths, fallback position. These ARE the cipher dsh-agent context — routing decisions read DATA, not baked prompts (fresh, compact, citable)
3. **Layer 3** — `voice-router` dsh agent (Archon workflow, a2a-exposed, inputs:/returns: signature = tool schema) + Hermes desktop plugin

## Portability rule (operator directive)
**The MCP contract is the ONLY public surface.** Every harness (Hermes desktop, Pinokio launcher, A0 skills) builds UI on the same MCP tools — no PMOVES-only protocol. Any MCP-capable harness gets voice for free.

## Ops
- Up: `docker compose -f pmoves/docker-compose.yml --profile orchestration --profile media up -d flute-gateway ultimate-tts-studio ffmpeg-whisper` (or make up-vibevoice for host-run provider)
- Config probe: `curl -fsS http://localhost:8055/v1/voice/config | jq .`
- Default provider: omnivoice (compose:4888); engine strengths: kokoro=fast-local, ultimate=quality/cloning, vibevoice=realtime-streaming, whisper=STT
