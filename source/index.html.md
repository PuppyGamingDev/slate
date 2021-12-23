---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - csharp

toc_footers:
  - <a href='https://portfolio.puppygaming.co.uk'>View my portfolio</a>

includes:
  - errors

search: true

code_clipboard: true

meta:
  - name: description
    content: Documentation for Puppy Gaming modifications
---

# Introduction

Welcome to the modifications documentation for Puppy Gaming. I am hoping to hold all the code snippets and written guide for modifications in my videos and also for some more bespoke modifications I have made.

# Super Multiplayer Shooter Template

## Host can restart game

This is a very simple modification to enable the game's host to restart the game once it's finished to avoid going back to the Lobby.

### GameManager.cs

> Look for lll and replace with
```csharp
public GameObject restartButton;
```
