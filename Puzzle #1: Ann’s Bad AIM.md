# Network Forensics Puzzle #1: Ann's Bad AIM — Walkthrough

## What we were given

A file called `evidence01.pcap`. This is a **packet capture** — a recording of every piece of network traffic that happened during a certain time. Our job was to look through it and figure out what Ann was doing.

![pcap file](Puzzle%20%231.%20Screenshots/pcap%20File.png)

## Step 1: Filter to one conversation using `tcp.stream`

In Wireshark, every TCP connection inside a pcap file gets its own ID number, starting at 0. This is called a **stream number**.

We used the filter:

```
tcp.stream eq 2
```

A `tcp.stream` filter shows only the packets that belong to **one single session**, instead of the whole file at once. Each session gets its own number, so by changing the number you can move through each separate conversation one at a time, in order, instead of scrolling through hundreds of unrelated packets.

This filter isolated the conversation between Ann's computer and the unknown laptop.

![tcp.stream filter applied](Puzzle%20%231.%20Screenshots/tcp.stream%20eq%202.png)

## Step 2: Read the conversation

With the filter applied, we right-clicked a packet and chose **Follow → TCP Stream**. This rebuilds the raw data into one readable block of text.

![Follow TCP stream](Puzzle%20%231.%20Screenshots/Follow%20TCP%20sream.png)

Inside it, we could clearly see:

```
Sec558user1 .... Here's the secret recipe... I just downloaded it from the
file server. Just copy to a thumb drive and you're good to go >:-)
```

![Conversation content](Puzzle%20%231.%20Screenshots/Conversation.png)

This one piece of text answers two questions at once.

## Question 1: What is the name of Ann's IM buddy?

**Answer: `Sec558user1`**

This name appears right next to the message content in the stream — it's the AIM username Ann was chatting with.

## Question 2: What was the first comment in the conversation?

**Answer:**
> "Here's the secret recipe... I just downloaded it from the file server. Just copy to a thumb drive and you're good to go"

This is the first real message visible once we followed the stream.

## Question 3: What is the name of the file Ann transferred?

**Answer: `recipe.docx`**

The filename appears directly in the stream data, right after the chat message, as part of the file transfer.

## Step 3: Extract the actual file using NetworkMiner

Reading the filename in the stream isn't enough — we needed the real file itself. Manually pulling file bytes out of raw packet data is difficult, so instead we used **NetworkMiner**, a tool made specifically for reconstructing files from network traffic automatically.

1. Opened NetworkMiner
2. Went to **File → Open** and loaded `evidence01.pcap`
3. NetworkMiner automatically scanned the whole file and rebuilt any files it found inside the traffic
4. Clicked the **Files** tab
5. Found `recipe.docx` listed there, fully reconstructed
6. Saved/copied it into our project folder

![NetworkMiner extraction](Puzzle%20%231.%20Screenshots/NetworkMiner.png)

This gave us a clean, complete copy of the actual file Ann sent — not just its name.

## Question 4: What is the magic number of the file (first four bytes)?

**Answer: `50 4B 03 04`**

Every file type starts with a specific set of bytes that identifies what kind of file it really is, no matter what it's named. This is called the **magic number**.

`50 4B 03 04` is the standard magic number for **ZIP files**. This makes sense because `.docx` files are not actually their own format — a `.docx` is really a ZIP archive on the inside, holding smaller files (text, styles, images) bundled together. That's why any Word, Excel, or PowerPoint file (`.docx`, `.xlsx`, `.pptx`) always begins with this same signature, regardless of what program made it.

## Step 4: Get the MD5 hash using HashMyFiles

To get a unique "fingerprint" of the extracted file, we used **HashMyFiles**, a small Windows tool that instantly calculates hash values for any file.

1. Opened HashMyFiles
2. Went to **File → Add Files**
3. Selected `recipe.docx`
4. The tool immediately displayed its MD5 value in a table

![HashMyFiles result](Puzzle%20%231.%20Screenshots/HashMyFiles.png)

## Question 5: What is the MD5sum of the file?

**Answer: `8350582774E1D4DBE1D61D64C89E0EA1`**

An MD5 hash works like a fingerprint — if even a single byte of the file changed, this value would come out completely different. This proves we have the exact, unmodified file, and lets anyone else verify they have the same one.

## Question 6: What is the secret recipe?

To read this, we opened the extracted `recipe.docx` file directly.

![Recipe document opened](Puzzle%20%231.%20Screenshots/Recipe.docx.png)

**Answer:**
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

This turned out to be a joke, not a real company secret. Pouring sugar water into a car's gas tank is a well-known myth about "sabotaging" an engine — so the "secret recipe" Ann was leaking was actually harmless.

## Summary Table

| # | Question | Answer |
|---|----------|--------|
| 1 | Ann's IM buddy | Sec558user1 |
| 2 | First comment | "Here's the secret recipe... I just downloaded it from the file server. Just copy to a thumb drive and you're good to go" |
| 3 | File name | recipe.docx |
| 4 | Magic number | 50 4B 03 04 |
| 5 | MD5sum | 8350582774E1D4DBE1D61D64C89E0EA1 |
| 6 | Secret recipe | A joke "recipe" for sabotaging a car engine with sugar water |

## Tools used

- **Wireshark** — to open the pcap file, filter to one conversation, and read the chat content
- **NetworkMiner** — to automatically reconstruct the transferred file from the raw traffic
- **HashMyFiles** — to calculate the file's MD5 hash

## Why this matters

This puzzle shows a real skill used in digital forensics: pulling a full story (who talked to who, what they said, what they sent) out of raw network traffic using only a packet capture. It also shows why file signatures (magic numbers) matter more than file names — a file can be renamed to anything, but its actual starting bytes always reveal what it truly is.
