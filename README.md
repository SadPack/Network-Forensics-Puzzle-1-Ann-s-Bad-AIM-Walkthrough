# Network Forensics Puzzle #1: Ann's Bad AIM — Walkthrough

## Overview

This walkthrough documents the process of analyzing a packet capture file to reconstruct a user's activity, extract a transferred file, and verify its integrity using standard forensic tools.

## Step 1: Isolate the Conversation Using a Stream Filter

Every TCP connection inside a pcap file is assigned its own ID number by Wireshark, starting at 0. This is called a stream number. Filtering by stream number lets you isolate one single session out 
of potentially hundreds of unrelated packets.

We applied the following filter in Wireshark:

```
tcp.stream eq 2
```

This filter isolated the conversation between Ann's computer and the unknown laptop she was communicating with.

## Step 2: Reconstruct and Read the Conversation

With the filter applied, we right-clicked a packet and selected Follow TCP Stream. This rebuilds the raw packet data into one continuous, readable block of text. Inside it, the following message 
content was visible:

```
Sec558user1 .... Here's the secret recipe... I just downloaded it from the
file server. Just copy to a thumb drive and you're good to go >:-)
```

This single piece of text answered two of the puzzle's questions at once who Ann was talking to, and what was said.

### Question 1: What is the name of Ann's IM buddy?

The username appears directly next to the message content in the stream.

```
Sec558user1
```

### Question 2: What was the first comment in the conversation?

The first real message visible once the stream was followed:

```
Here's the secret recipe... I just downloaded it from the file server. Just copy to a thumb drive and you're good to go
```

### Question 3: What is the name of the file Ann transferred?

The filename appears directly in the stream data, immediately after the chat message, as part of the file transfer.

```
recipe.docx
```

## Step 3: Extract the Actual File Using NetworkMiner

Reading the filename inside the stream isn't the same as having the file itself. Manually carving file bytes out of raw packet data is difficult and error-prone, so instead we used NetworkMiner, 
a tool built specifically for automatically reconstructing files from network traffic.

The process follows: we opened NetworkMiner, went to File Open, and loaded `evidence01.pcap`. NetworkMiner automatically scanned the capture and rebuilt any files it detected inside the 
traffic. We then clicked the Files tab, located `recipe.docx` fully reconstructed in the list, and saved a copy of it into our project folder.

This gave us a clean, complete copy of the actual file Ann sent, rather than just a filename referenced in a chat log.

### Question 4: What is the magic number of the file (first four bytes)?

Every file type begins with a specific sequence of bytes that identifies what kind of file it actually is, regardless of what it's named. This is called the magic number.

```
50 4B 03 04
```

This is the standard magic number for ZIP files. That makes sense here because a `.docx` file isn't really its own format on the inside — it's a ZIP archive containing smaller files such as 
text, styles, and images, all bundled together. This is why any Word, Excel, or PowerPoint file (`.docx`, `.xlsx`, `.pptx`) always begins with this same signature, no matter what program created 
it or what the file is named.

## Step 4: Get the MD5 Hash Using HashMyFiles

To generate a unique fingerprint of the extracted file, we used HashMyFiles, a lightweight Windows utility that instantly calculates hash values for any file. We opened HashMyFiles, went to File
Add Files, and selected `recipe.docx`. The tool immediately displayed its MD5 value in a results table.

### Question 5: What is the MD5sum of the file?

```
8350582774E1D4DBE1D61D64C89E0EA1
```

An MD5 hash works like a fingerprint: if even a single byte of the file changed, this value would come out completely different. Recording this hash proves we have the exact, unmodified file, 
and lets anyone else verify they're working from the same copy.

## Step 5: Read the Extracted Document

To answer the final question, we opened the extracted `recipe.docx` file directly. It opened normally in Microsoft Word, confirming it was a genuine, valid document rather than a corrupted or 
malicious file.

### Question 6: What is the secret recipe?

```
Recipe for Disaster:
1 serving
Ingredients:
4 cups sugar
2 cups water
In a medium saucepan, bring the water to a boil. Add sugar. Stir gently over
low heat until sugar is fully dissolved. Remove the saucepan from heat. Allow
to cool completely. Pour into gas tank. Repeat as necessary.
```

| # | Question | Answer |
|---|----------|--------|
| 1 | Ann's IM buddy | Sec558user1 |
| 2 | First comment | "Here's the secret recipe... I just downloaded it from the file server. Just copy to a thumb drive and you're good to go" |
| 3 | File name | recipe.docx |
| 4 | Magic number | 50 4B 03 04 |
| 5 | MD5sum | 8350582774E1D4DBE1D61D64C89E0EA1 |
| 6 | Secret recipe | A joke "recipe" for sabotaging a car engine with sugar water |

## Tools Used

- **Wireshark** — used to open the pcap file, filter to a single conversation, and read the reconstructed chat content
- **NetworkMiner** — used to automatically reconstruct the transferred file from the raw traffic
- **HashMyFiles** — used to calculate the file's MD5 hash
