---
layout: post
title: "Spotify Reshuffle: Finally, a Truly Random Shuffle!"
date: 2025-01-15
categories: [software, music, rust, cli-tools]
tags: [spotify, rust, cli, music, shuffle, randomization]
excerpt: "Tired of Spotify's fake shuffle? I was too. That's why I built spotify-reshuffle—a tool that gives you truly random playlist shuffling."
---

Ever hit shuffle on Spotify and feel like you're stuck in a loop? I did, and it drove me nuts. After hearing the same songs repeat over and over, I finally got fed up and decided to build my own solution.

**spotify-reshuffle** is a command-line tool built in Rust that actually gives you truly random shuffling of your Spotify playlists and liked songs.

<p align="center"><img width="600" alt="Spotify Reshuffle" src="{{ '/assets/images/spotifyreshuffle.png' | relative_url }}" /></p>

## The Problem with Spotify's "Random" Shuffle

Let's be honest—Spotify's shuffle isn't really random. I was initially frustrated to feel that the random wasn't true random. Turns out, I'm not alone in this frustration.

Many users have complained about this issue across various platforms. For instance, [Reddit users discussing Spotify shuffle issues](https://www.reddit.com/r/spotify/search/?q=shuffle%20not%20random&restrict_sr=1&sort=relevance&t=all) highlight the common complaints.

The problem is that Spotify's algorithm is designed to "enhance" the listening experience by avoiding consecutive songs from the same artist or album, which creates patterns that feel anything but random. Users frequently complain about:
- Repetitive patterns during playback
- Predictable song sequences  
- The feeling that shuffle isn't actually random

## The Solution: spotify-reshuffle

spotify-reshuffle uses true randomization that actually gives every song an equal chance of being played—no more predictable patterns or fake randomness.

It combines tracks from your playlists and liked songs, removes duplicates, and creates a new playlist with truly random order. That's it.

### Getting Started

```bash
# Download the latest release
curl -L -o spotify-reshuffle.tar.gz https://github.com/ncolomer/spotify-reshuffle/releases/latest/download/spotify-reshuffle-linux-amd64.tar.gz
tar -xzf spotify-reshuffle.tar.gz
chmod +x spotify-reshuffle

# Set up your Spotify credentials
export SPOTIFY_CLIENT_ID="your_client_id_here"
export SPOTIFY_CLIENT_SECRET="your_client_secret_here"

# Create a truly random mix
spotify-reshuffle --target-playlist-name "My Random Mix" --include-liked
```

## Check It Out

All the code is open source and available on GitHub:
- **Source Code**: [github.com/ncolomer/spotify-reshuffle](https://github.com/ncolomer/spotify-reshuffle)
- **Releases**: Download pre-built binaries for Linux (x86_64 and ARM64), macOS (Intel and Apple Silicon)

The repository has everything you need:
- Complete Rust source code
- Cross-platform build support
- Comprehensive documentation
- CI/CD workflows for automated releases

---

*Finally, a shuffle that's actually random! Give spotify-reshuffle a try and rediscover your music library without the predictable patterns that make Spotify's shuffle so frustrating.*
