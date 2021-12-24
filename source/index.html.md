---
title: PuppyGaming Docs

language_tabs: # must be one of https://git.io/vQNgJ
  - csharp

toc_footers:
  - <a href='https://portfolio.puppygaming.co.uk'>View my portfolio</a>

includes:
  - errors

search: true

code_clipboard: true

meta:
  - name: Puppy Gaming
    content: Documentation for Puppy Gaming modifications
---

# Introduction

Welcome to the modifications documentation for Super Multiplayer Shooter Template. I am hoping to hold all the code snippets and written guide for modifications in my videos and also for some more bespoke modifications I have made.

# Host Can Restart Game

This is a very simple modification to enable the game's host to restart the game once it's finished to avoid going back to the Lobby.

## GameManager.cs

> Add this reference

```csharp
public GameObject restartButton;
```

> Add this to void Start()

```csharp
if (PhotonNetwork.IsMasterClient)
{
  restart.Button.SetActive(true);
}
```

> Add this to void Awake()

```csharp
PhotonNetwork.AutomaticallySyncScene = true;
```

> Add these Restart methods

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
This is the only script that needs editing for this modification.

## Canvas in Game Scene

In the GameOverPanel gameobject, add a button and set the gameobject as inactive.
Drag the Managers Gameobject into the new button's OnClick() and set the function to GameManager>RestartButton.


# Cosmetics with CBS Inventory

This allows you to link the current SMST Cosmetic items to CBS Items and use CBS for the users cosmetics inventory.

## CharacterCustomizer.cs

> At the top of the script add these

```csharp
using System.Linq;
using CBS;
```

> Add these references

```csharp
public List<CosmeticItemData> items;
public CosmeticItemData[] allCosmetics;
private ICBSInventory InventoryModule { get; set; }
```

> Replace *SampleInventory.instance.items.Length* with

```csharp
items.Count
```

> Ad also replace *SampleInventory.instance.items[i]* with

```csharp
items[i]
```

> At the beginning of void Start(), add these

```csharp
InventoryModule = CBSModule.Get<CBSInventory>();
InventoryModule.GetInventory(OnGetInventory);
```

> At the start of void Open(), add this

```csharp
if (curSlots.Count < items.Count)
{
  curSlots= new List<InventorySlot>();
  for (int i = 0; i< items.Count, i++)
  {
    InventorySlot s = Instantiate(slotPrefab, slotHandler);
    s.cc = this;
    curSlots.Add(s);
  }
  RefreshSlots();
}
```

> Then add this method

```csharp
private void OnGetInventory(GetInventoryResult result)
{
  if (result.IsSuccess)
  {
    var allItems = result.AllItems;
    foreach (CBSInventoryItem item in result.AllItems)
    {
      foreach (CosmeticItemData i in allCosmetics)
      {
        if (i.cbsItemID == item.ID)
        {
          if (!items.Contains(i))
          {
            int addLocation = 0;
            while (addLocation < items.Count)
            {
              addLocation++;
            }
            items.Add(i);
            Debug.Log("Found item from CBS ID; " + item.ID + " which mathces item " + i.name);
          }
        }
      }
    }
  }
}
```

In this script we are chaging where we get the list of SMST Cosmetic Items, creating a new list which will be what the cosmetics window uses to populate and then comparing to Cosmetic Items available against the items the player has in their CBS Inventory and add it to our new list

## CosmeticItemData.cs

> Add this variable

```csharp
public string cbsItemID;
```

Here we are just adding a reference to the cosmetic items counterpart in the CBS Items. This item id will be what we set the CBS Item ID as. So give each of your Cosmetic Items a memorable ID like "hat1"

## CBS Configurator

```csharp
// no code on this section
```

In the Configurator, go to Items and create a new item to match one of your current SMST Cosmetics, and make sure to set the Item ID then same as what you added as the items cbsItemID.

## Main Menu Scene

```csharp
// no code in this section
```

On the *Managers* GameObject, add all your SMST cosmetics to the CharacterCustomizer component under the *All Items* list. Also make sure you items are still updated in the *ItemLibrary* GameObject as per normal when creating new SMST Cosmetics.

## How to test

```csharp
// no code in this section
```

If you wish to test this out, you can go to your PlayFab Dashboard, find your player and go to Inventory. You can then grant your player an item, run the game and open up your cosmetics page and you should now see the items you own with CBS.

# Line Render while aiming joystick

This will only work while using mobile joysticks and will display a line from the player in the aiming direction

## PlayerController.cs

> Add these references 

```csharp
public GameObject lineStart;
public GameObject lineExit;
```

> In *void Update()* directly underneath where it says *HandleInputs();* add this

```csharp
if (gm.controlsManager.shootStick.isHolding)
{
  LineRenderer lineRenderer = GetComponent<LineRenderer>();
  lineRenderer.enabled = true;
  lineRenderer.positionCount = 2;
  lineRenderer.SetPosition(0, lineStart.transform.position);
  lineRenderer.SetPosition(1, lineExit.transform.position);
}
else
{
  LineRenderer lineRenderer = GetComponent<LineRenderer>();
  lineRenderer.enabled = false;
}
```
Here we are stating that when we are holding the shooting joystick down, we are activation the Line Renderer component between the start and exit points that we are going to set.

## Player Prefab

```csharp
// no code in this section
```

Add a Line Renderer component to your Player prefab and untick it so by default it will be inactive then inside the WeaponHandler Gameobject, add 2 empty gameobjects and call them LineStart & LineExit. Set the transform of LineStart to (0, 0, 0) and the transform of LineExit to (10, 0, 0). To edit the appearance you can tick the component to make it active again and change the component's Material, then when finished just make sure it's unticked again.

# Shoot on Joystick release

With this, it will shoot when you release the shooting joystick. This is handy for Sniper type weapons where you can hold it to aim and then release to shoot.

## Joystick.cs

> Add this reference

```csharp
public bool canShoot;
```

> In *OnPointerUp()* add

```csharp
canShoot = true;
```
This just to say that when we release the touch on the joystick, we are allowed to shoot.

## ControlsManager.cs

> Add this reference

```csharp
public bool canShootNow;
```

> In *LateUpdate()*, comment out the current part with *//* and add this below

```csharp
shoot = mobileControls ? canShootNow : Input.GetButton("Fire1");
```

> In *Update()* add this

```csharp
canShootNow = shootStick.canShoot;
```

We are just modifying the current 'shoot' bool which the PlayerController script will use for shooting.

## PlayerController.cs

> In *public void Shoot()*, add this at the end

```csharp
GameManager.instance.controlsManager.shootStick.canShoot = false;
```

This is setting the canShoot bool back to false after we have taken a shot, otherwise we would continiously shoot after we release the joystick.