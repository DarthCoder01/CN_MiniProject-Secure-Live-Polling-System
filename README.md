# Secure Live Polling System

A real-time voting application built with low-level UDP socket programming. All traffic is encrypted using **DTLS (Datagram Transport Layer Security)**. Supports multiple concurrent voters, custom binary packets, duplicate detection, and live result broadcasting.

---

## Architecture

```
Client 1 ──┐
Client 2 ──┼──► UDP + DTLS ──► Server (0.0.0.0:5005)
Client N ──┘                        │
                               ┌────▼─────┐
                               │  Thread  │  (one per client)
                               │  + Stats │
                               └──────────┘
```

| File | Role |
|---|---|
| `server.py` | Accepts DTLS connections, validates votes, tracks stats |
| `client.py` | Voting UI, sends encrypted packets, shows live results |
| `packet.py` | 19-byte binary packet — encode, decode, checksum |
| `stats.py` | Votes, duplicates, latency, throughput tracking |
| `stress_test.py` | Simulates N concurrent voters for performance testing |

---

## Packet Format

```
| voter_id (4B) | seq_num (4B) | candidate_id (1B) | timestamp (8B) | checksum (2B) |
                                                              Total = 19 bytes
```

---

## Setup

**1. Install dependency** (all machines):
```bash
pip install python3-dtls
```

**2. Generate certificate** (server machine only):
```bash
openssl req -new -newkey rsa:2048 -days 365 -nodes -x509 -keyout server.key -out server.crt -subj "/CN=polling-server"
```

---

## Usage

**Start server:**
```bash
python3 server.py
```

**Configure client** — open `client.py` and set:
```python
SERVER_HOST = "192.168.x.x"  # server's IP address
```

**Start client:**
```bash
python3 client.py
```
Follow the on-screen menu to vote. Live results appear every 5 seconds automatically.

**Run performance test:**
```bash
python3 stress_test.py --host 192.168.x.x --voters 20
```

---

## Key Features

- **DTLS encryption** — all votes encrypted over UDP end-to-end
- **Duplicate detection** — each voter ID can only vote once
- **Checksum validation** — corrupted packets are detected and dropped
- **ACK + retry** — client retries up to 3 times if no acknowledgment received
- **Live results** — background thread polls server every 5 seconds
- **Performance metrics** — latency, throughput, and packet loss tracked server-side
