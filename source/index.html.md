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

```csharp
public GameObject restartButton;
```
Add this GameObject reference to the script.

```csharp
if (PhotonNetwork.IsMasterClient)
{
  restart.Button.SetActive(true);
}
```

In void Start(), add this


```csharp
PhotonNetwork.AutomaticallySyncScene = true;
```

In void Awake(), add this

```csharp
public void RestartButton()
{
  photonView.RPC("Restart", RpcTarget.All);
}
[PunRPC]
void Restart()
{
  ExitGames.Client.Photon.Hashtable table = new ExitGames.Client.Photon.Hashtable();
  table.Clear();
  table.Add("kills", 0);
  table.Add("deaths", 0);
  table.Add("otherscore", 0);
  PhotonNetwork.LocalPlayer.SetCustomProperties(table);
  ui.UpdateBoards();
  Debug.Log("Stats Reset");
  PhotonNetwork.LoadLevel("Game");
}
```

Add these methodsand save the script

### Canvas in Game Scene



In the GameOverPanel gameobject, add a button and set the gameobject as inactive.
Drag the Managers Gameobject into the new button's OnClick() and set the function to GameManager>RestartButton.