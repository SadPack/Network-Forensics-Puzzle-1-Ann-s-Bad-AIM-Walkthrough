# Network Forensics Puzzle 

## Overview

This walkthrough documents the process of analyzing a packet capture file to reconstruct a user's activity, extract a transferred file, and verify its integrity using standard forensic tools.

## Tools Used

- **Wireshark** — used to open the pcap file, filter to a single conversation, and read the reconstructed chat content
- **NetworkMiner** — used to automatically reconstruct the transferred file from the raw traffic
- **HashMyFiles** — used to calculate the file's MD5 hash
