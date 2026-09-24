# Wi-Fi Latency Root Cause Investigation

## Goal
Diagnose why gateway ping tests were showing packet loss and inconsistent 
latency on one Wi-Fi network, and determine whether the cause was the 
laptop or the network itself.

## Method
Used Windows `ping` and PowerShell `Test-Connection` to test round-trip 
time (RTT) and packet loss against:
- The default gateway (router) on the problematic network
- A known-reliable external host (Google DNS-adjacent test targets, 
  scanme.nmap.org)
- The same gateway test repeated on a second, different Wi-Fi network, 
  using the same laptop, as a control comparison

## Findings

| Test | Network | Target | Loss | RTT Range | Average | Verdict |
|------|---------|--------|------|-----------|---------|---------|
| Gateway ping (2nd test) | Network A | 10.21.58.2 (gateway) | 0% | 6ms - 836ms | 417ms | Very unhealthy (extreme spike) |
| Gateway ping (control) | Network B | 10.83.168.250 (gateway) | 0% | 5ms - 10ms | 7ms | Healthy |
| Remote host ping | Network A | 45.33.32.156 (scanme.nmap.org) | 0% | 232ms - 251ms | 242ms | Healthy (expected distance latency) |

## Investigation

A healthy local gateway ping should be consistently low (1-5ms) with 0% 
loss, since it's only one hop away. Network A instead showed both packet 
loss and RTTs ranging as high as 836ms — far worse than even a ping to a 
remote server roughly 200ms away (scanme.nmap.org).

To isolate whether this was a laptop hardware/driver issue or a network 
issue, the same laptop was connected to a second, unrelated Wi-Fi network 
(Network B) and the same gateway ping test was repeated.

**Result:** Network B produced a clean, consistent result (5-10ms, 0% loss) 
— the kind of result expected for a healthy local network.

## Conclusion

Since the same laptop performed well on Network B, the laptop's Wi-Fi 
hardware and drivers are unlikely to be the root cause. The issue appears 
to be specific to Network A — most likely Wi-Fi interference, channel 
congestion, or router load, rather than a laptop-side problem.

## Limitations / what would fully confirm this

This investigation strongly suggests a network-side issue but doesn't 
fully prove it. To rule out the laptop with full certainty, the next 
steps would be:
- Testing a second device on Network A to see if it shows the same 
  erratic behavior
- Reconnecting to Network A later to see if the issue is persistent or 
  was transient congestion
- Checking Network A's Wi-Fi band (2.4GHz vs 5GHz) and channel congestion 
  directly from the router's admin page

## Tools used
- Windows `ping` / PowerShell `Test-Connection`
- Wireshark (for packet-level verification of ICMP traffic)
