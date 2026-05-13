# SMF Patches

Applied on top of Open5GS at the tag pinned in `upstream-versions.json`.

---

## 0001 — session-info endpoint

**File:** `0001-session-info-endpoint.patch`

Adds `GET /session-info` on the SMF metrics port (`:9090`). Returns all active PDU sessions as JSON without requiring log parsing.

### Modified files

| File | Change |
|------|--------|
| `lib/metrics/context.h` | Adds `ogs_metrics_set_custom_url_handler()` API |
| `lib/metrics/prometheus/context.c` | Hooks custom handler into MHD request dispatch |
| `lib/metrics/void/context.c` | No-op stub for builds without Prometheus |
| `src/smf/init.c` | Registers handler at SMF startup |
| `src/smf/meson.build` | Adds `session-info.c/h` to build |
| `src/smf/session-info.c` | Handler implementation (new file) |
| `src/smf/session-info.h` | Handler declaration (new file) |

### Endpoint

```
GET http://<smf-addr>:9090/session-info
```

### Response

```json
{
  "sessions": [
    {
      "imsi": "001010123456786",
      "dnn": "internet",
      "ipv4": "10.45.0.6",
      "ipv6": "",
      "snssai": { "sst": 1, "sd": "000001" },
      "up_cnx_state": "ACTIVATED"
    }
  ]
}
```

### Fields

| Field | Source | Notes |
|-------|--------|-------|
| `imsi` | `smf_ue->supi` | `imsi-` prefix stripped |
| `dnn` | `sess->session.name` | Data Network Name |
| `ipv4` | `sess->ipv4->addr[0]` | UPF-assigned via PFCP; empty string if IPv4 not allocated |
| `ipv6` | `sess->ipv6->addr` | Empty string if IPv6 not allocated |
| `snssai` | `sess->s_nssai` | Slice identifier |
| `up_cnx_state` | `sess->up_cnx_state` | `ACTIVATED`, `DEACTIVATED`, `ACTIVATING`, or `SUSPENDED` |

### Implementation notes

- Handler registered via `ogs_metrics_set_custom_url_handler()` — plugs into the existing MHD server, no new port or thread.
- Response buffer allocated with `ogs_msprintf` (uses `ogs_malloc`). MHD copy mode (`MHD_RESPMEM_MUST_COPY`) used to avoid allocator mismatch with libc `free()`.
- Read-only access to in-memory session state. No protocol behavior affected.
