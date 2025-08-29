# xenterface Framework Documentation

`xenterface` is a **UI management framework** for **Roblox** that lets you easily create **interactive**, **animated user interfaces**. It consists of two intergrated systems:

- `xenterface`: A page/tab navigation controller that manages UI state, page switching, and element organization
- `xenimation`: A utility-class based animation sub-framework that provides declarative, state-based animations similar to [Tailwind CSS](https://tailwindcss.com/)

Go from **hundreds of lines of code**, to **simple** tag and attribute configurations with **minimal code**.

# Contents

- [Why?](#why)
- [Code Comparison](#code-comparison)
  - [Method 1: Without xenterface](#without-xenterface)
  - [Method 2: With xenterface](#with-xenterface)
  - [Method 3: With xenterface + xenimation](#with-xenterface-and-xenimation)
  - [Time Comparisons](#time-comparison)
- [xenterface Documentation](#xenterface-documentation)
  - [Setup](#xenterface-setup)
  - [Examples](#xenterface-examples)
  - [API Reference](#xenterface-api-reference)
    - [Core Module](#core-module)
    - [Controller Object](#controller-object)
    - [Hover Object](#hover-object)
    - [Tags](#tags)
    - [Attributes Reference](#attributes-reference)
- [xenimation Documentation](#xenimation-documentation)
  - [Setup](#xenimation-setup)
  - [Multiple Animations](#multiple-animations-per-guiobject)
  - [Animation Presets](#animation-presets)
  - [Manual Animation Control](#manual-animation-control)
  - [Xenterface Integration](#integration-with-xenterface-navigation)
    - [Tab + Page](#tab-and-page-integration)
    - [Hover](#hover-integration)
    - [Configuration References Multiple Animations](#advanced-configuration-references-multiple-animations)
    - [Combined Tab + Hover](#combined-tab--hover)
    - [Conflicts & Edge Cases](#property-conflicts-and-edge-cases)
    - [Shortcuts](#quick-animation-shortcuts)
    - [Advanced Features](#advanced-features)
      - [Initial](#the-initial-keyword)
      - [Relative Operations](#relative-operations)
      - [Utility Class Reference](#utility-class-reference)


# Why?

Traditional Roblox UI development requires:
- Manual TweenService setup for every animation
- Complex state management for page switching
- Repetitive code for hover effects and transitions
- Careful connection management to prevent memory leaks
- Synchronization between related UI elements

xenterface solves these problems by providing:
- **Zero-code animations** through attributes
- **Automatic state management** for pages and tabs
- **Built-in memory management** and connection cleanup
- **Declarative syntax** that's easy to read and maintain
- **Reusable presets** for consistent design patterns

# Code Comparison

### Without xenterface
~120+ lines of code
<details>
<summary>Method 1 Code</summary>

```lua
-- Vanilla implementation: ~120+ lines of code
local TweenService = game:GetService("TweenService")
local Players = game:GetService("Players")
local player = Players.LocalPlayer
local gui = player.PlayerGui:WaitForChild("SettingsGui")

-- Get all UI elements
local tabs = {
    General = gui.TabBar.GeneralTab,
    Audio = gui.TabBar.AudioTab,
    Graphics = gui.TabBar.GraphicsTab
}

local pages = {
    General = gui.Pages.GeneralPage,
    Audio = gui.Pages.AudioPage,
    Graphics = gui.Pages.GraphicsPage
}

-- Track active tweens to prevent conflicts
local activeTweens = {}
local currentPage = "General"

-- Store original sizes for hover effect
local originalSizes = {}
for name, tab in tabs do
    originalSizes[name] = UDim2.new(0, 200, 0, 40)
end

-- Function to animate tab selection
local function selectTab(tabName)
    -- Cancel existing tweens
    for _, tween in activeTweens do
        tween:Cancel()
    end
    activeTweens = {}
    
    -- Animate tabs
    for name, tab in tabs do
        if name == tabName then
            -- SELECTED TAB: Scale up + Blue color + White text
            local scaleTween = TweenService:Create(tab, TweenInfo.new(0.2, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
                Size = UDim2.new(0, 210, 0, 42)
            })
            local colorTween = TweenService:Create(tab, TweenInfo.new(0.2, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
                BackgroundColor3 = Color3.fromRGB(88, 101, 242)
            })
            local textTween = TweenService:Create(tab.TextLabel, TweenInfo.new(0.2, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
                TextColor3 = Color3.fromRGB(255, 255, 255)
            })
            scaleTween:Play()
            colorTween:Play()
            textTween:Play()
            table.insert(activeTweens, scaleTween)
            table.insert(activeTweens, colorTween)
            table.insert(activeTweens, textTween)
        else
            -- UNSELECTED TAB: Normal size + Gray color + Gray text
            local scaleTween = TweenService:Create(tab, TweenInfo.new(0.1, Enum.EasingStyle.Linear, Enum.EasingDirection.Out), {
                Size = originalSizes[name]
            })
            local colorTween = TweenService:Create(tab, TweenInfo.new(0.1, Enum.EasingStyle.Linear, Enum.EasingDirection.Out), {
                BackgroundColor3 = Color3.fromRGB(50, 50, 50)
            })
            local textTween = TweenService:Create(tab.TextLabel, TweenInfo.new(0.1, Enum.EasingStyle.Linear, Enum.EasingDirection.Out), {
                TextColor3 = Color3.fromRGB(200, 200, 200)
            })
            scaleTween:Play()
            colorTween:Play()
            textTween:Play()
            table.insert(activeTweens, scaleTween)
            table.insert(activeTweens, colorTween)
            table.insert(activeTweens, textTween)
        end
    end
    
    -- Animate pages
    for name, page in pages do
        if name == tabName then
            page.Visible = true
            local tween = TweenService:Create(page, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
                Position = UDim2.new(0.5, 0, 0.5, 0),
                GroupTransparency = 0
            })
            tween:Play()
            table.insert(activeTweens, tween)
        else
            local tween = TweenService:Create(page, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
                Position = UDim2.new(0.5, 0, 0.5, 60),
                GroupTransparency = 1
            })
            tween:Play()
            tween.Completed:Connect(function()
                page.Visible = false
            end)
            table.insert(activeTweens, tween)
        end
    end
    
    currentPage = tabName
end

-- Add hover effects
for name, tab in tabs do
    local hoverIn = TweenService:Create(tab, TweenInfo.new(0.2, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
        Size = UDim2.new(0, 210, 0, 42)
    })
    local hoverOut = TweenService:Create(tab, TweenInfo.new(0.1, Enum.EasingStyle.Linear, Enum.EasingDirection.Out), {
        Size = originalSizes[name]
    })
    
    tab.MouseEnter:Connect(function()
        if currentPage ~= name then hoverIn:Play() end
    end)
    tab.MouseLeave:Connect(function()
        if currentPage ~= name then hoverOut:Play() end
    end)
    
    tab.MouseButton1Click:Connect(function()
        selectTab(name)
    end)
end

-- Initialize
selectTab("General")
```
</details>

### With xenterface:
~50 lines of code
<details>
<summary>Method 2 Code</summary>
  
```lua
-- xenterface implementation: ~50 lines of code
local xenterface = require(game.ReplicatedStorage.xenterface)
local TweenService = game:GetService("TweenService")

-- Get the controller for our settings menu
local controller = xenterface:GetController("SettingsMenu")

-- Track current selected tab for hover logic
local currentSelectedTab = nil

-- Define what happens when tabs/pages are selected
controller.SelectedTab = function(tab)
    -- SELECTED TAB: Scale up + Blue color + White text
    currentSelectedTab = tab
    
    TweenService:Create(tab, TweenInfo.new(0.2, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
        Size = UDim2.new(0, 210, 0, 42),  -- 1.05x scale
        BackgroundColor3 = Color3.fromRGB(88, 101, 242)  -- Blue
    }):Play()
    
    TweenService:Create(tab.TextLabel, TweenInfo.new(0.2, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
        TextColor3 = Color3.fromRGB(255, 255, 255)  -- White
    }):Play()
end

controller.UnselectedTab = function(tab)
    -- UNSELECTED TAB: Normal size + Gray color + Gray text
    TweenService:Create(tab, TweenInfo.new(0.1, Enum.EasingStyle.Linear, Enum.EasingDirection.Out), {
        Size = UDim2.new(0, 200, 0, 40),
        BackgroundColor3 = Color3.fromRGB(50, 50, 50)  -- Gray
    }):Play()
    
    TweenService:Create(tab.TextLabel, TweenInfo.new(0.1, Enum.EasingStyle.Linear, Enum.EasingDirection.Out), {
        TextColor3 = Color3.fromRGB(200, 200, 200)  -- Light gray
    }):Play()
end

controller.SelectedPage = function(page)
    -- SELECTED PAGE: Fade in + Move to center
    page.Visible = true
    TweenService:Create(page, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
        Position = UDim2.new(0.5, 0, 0.5, 0),
        GroupTransparency = 0
    }):Play()
end

controller.UnselectedPage = function(page)
    -- UNSELECTED PAGE: Fade out + Move down
    local tween = TweenService:Create(page, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
        Position = UDim2.new(0.5, 0, 0.5, 60),
        GroupTransparency = 1
    })
    tween:Play()
    tween.Completed:Connect(function()
        page.Visible = false
    end)
end

-- Set up hover effects using xenterface's Hover API
-- Assuming tabs are tagged with "Hover" in Studio
local tabs = {"GeneralTab", "AudioTab", "GraphicsTab"}
for _, tabName in tabs do
    local tab = gui.TabBar[tabName] -- Assuming gui is accessible
    local hover = xenterface:GetHover(tab)
    if hover then
        -- HOVER: Scale to 1.05x (only if not selected)
        hover.EnterFunction = function(guiObject)
            if guiObject ~= currentSelectedTab then
                TweenService:Create(guiObject, TweenInfo.new(0.2, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
                    Size = UDim2.new(0, 210, 0, 42)  -- 1.05x scale
                }):Play()
            end
        end
        
        hover.LeaveFunction = function(guiObject)
            if guiObject ~= currentSelectedTab then
                TweenService:Create(guiObject, TweenInfo.new(0.1, Enum.EasingStyle.Linear, Enum.EasingDirection.Out), {
                    Size = UDim2.new(0, 200, 0, 40)  -- Normal size
                }):Play()
            end
        end
    end
end

-- Tags and attributes set in Studio:
-- Each tab needs: Tags = "Tab", "Hover" | PageGroup = "SettingsMenu" | PageId = "General"/"Audio"/"Graphics"
-- Each page needs: Tag = "Page" | PageGroup = "SettingsMenu" | PageId = "General"/"Audio"/"Graphics"
```
</details>

### With xenterface and xenimation:
12 lines of code with studio and preset setup
<details>
<summary>Method 3</summary>
  
```lua
-- Complete implementation: Just 1 line!
local xenterface = require(game.ReplicatedStorage.xenterface)
```

### One-time Preset Setup (in xenimation/Presets module) - 11 lines:
```lua
return {
    -- Tab selection animations (matching vanilla exactly)
    tabSelected = {
        Active = "scale-1.05 bgc-blue-0.8 tc-white-1 t-0.2 back out",
        Inactive = "initial bgc-gray-0.2 tc-gray-0.8 t-0.1 linear"
    },
    
    -- Tab hover animations (matching vanilla exactly)  
    tabHoverEffect = {
        Active = "scale-1.05 t-0.2 back out",
        Inactive = "initial t-0.1 linear"
    },
    
    -- Page visibility animations (matching vanilla exactly)
    pageVisibility = {
        Active = "pos-c gt-0 t-0.3 quad out",
        Inactive = "pos-c-60yo gt-1 t-0.2 quad out"
    }
}
```

### Studio Configuration:

**For each tab (e.g., GeneralTab):**
1. **The Tab itself** gets:
   - Tags: `Tab`, `Animation`
   - PageGroup: "SettingsMenu"
   - PageId: "General"
   - Preset: "tabSelected"

2. **Add a Configuration child** named "HoverAnimation":
   - Tags: `Animation`, `Hover`
   - Preset: "tabHoverEffect"

**For each page:**
- Tags: `Page`, `Animation`
- PageGroup: "SettingsMenu"
- PageId: [Their respective name]
- Preset: "pageVisibility"

**Total lines of code: 1 line (main) + 11 lines (presets) = 12 lines**

</details>

## Time Comparison

| Method | Total Implementation Time |
|--------|--------------------------|
| **Without xenterface (120 lines)** | 60-90 minutes |
| **With xenterface (50 lines)** | 20-30 minutes |
| **With xenterface + xenimation (12 lines)** | 3-5 minutes |

The progression shows a dramatic reduction in development time—from over an hour of coding and debugging to just a few minutes of configuration.

### The Result:
All three methods produce **EXACTLY** the same UI behavior:
- ✅ Tabs scale to 1.05x when selected (with blue background)
- ✅ Selected tabs turn blue with white text
- ✅ Unselected tabs are gray
- ✅ Hover effect scales tabs to 1.05x (without color change)
- ✅ Pages fade in/out with position animation
- ✅ Smooth transitions with proper easing curves

# Xenterface Documentation

## Xenterface Setup

### Step 1: Install xenterface
Place the `xenterface` module in `ReplicatedStorage` (or wherever your modules live).

### Step 2: Require the framework
```lua
local xenterface = require(game.ReplicatedStorage.xenterface)
```

### Step 3: Tag your UI elements
In Roblox Studio, add tags and attributes to your GUI elements:

**For Tabs (clickable elements):**
- **Tag**: `Tab`
- **Attributes**: 
  - `PageGroup` = "YourGroupName" (string)
  - `PageId` = "YourPageId" (string)

**For Pages (content that shows/hides):**
- **Tag**: `Page` 
- **Attributes**:
  - `PageGroup` = "YourGroupName" (string)
  - `PageId` = "YourPageId" (string)

> **💡 Pro Tip: Multiple Tabs/Pages per GuiObject**
> 
> You can add multiple tab or page configurations to a single GuiObject using **Configuration objects** as children. This is useful when one GUI element needs to participate in multiple navigation systems.
>
> **Example Setup:**
> ```
> MyButton (GuiButton)
> ├── TabConfig1 (Configuration - Tag: "Tab", PageGroup: "MainMenu", PageId: "Home")  
> └── TabConfig2 (Configuration - Tag: "Tab", PageGroup: "Sidebar", PageId: "Quick")
> ```
>
> Now `MyButton` acts as both a "Home" tab for "MainMenu" group AND a "Quick" tab for "Sidebar" group. Clicking it will trigger both navigation systems simultaneously.

### Step 4: (Optional) Add controller logic
```lua
local controller = xenterface:GetController("YourGroupName")
controller.SelectedTab = function(tab) 
    -- What happens when a tab is selected
end
controller.SelectedPage = function(page)
    -- What happens when a page is shown
end
```

That's it! Clicking tabs will automatically switch pages.

# Xenterface Examples

### Example 1: Simple 2-Tab Navigation

**Studio Setup:**
```
MainFrame
├── TabBar
│   ├── HomeTab (Tag: "Tab", PageGroup: "Navigation", PageId: "Home")
│   └── SettingsTab (Tag: "Tab", PageGroup: "Navigation", PageId: "Settings")
└── ContentArea
    ├── HomePage (Tag: "Page", PageGroup: "Navigation", PageId: "Home")
    └── SettingsPage (Tag: "Page", PageGroup: "Navigation", PageId: "Settings")
```

**Script:**
```lua
local xenterface = require(game.ReplicatedStorage.xenterface)

-- Optional: Add visual feedback
local controller = xenterface:GetController("Navigation")

controller.SelectedTab = function(tab)
    tab.BackgroundColor3 = Color3.fromRGB(88, 101, 242) -- Blue when selected
end

controller.UnselectedTab = function(tab)
    tab.BackgroundColor3 = Color3.fromRGB(64, 64, 64) -- Gray when unselected
end

controller.SelectedPage = function(page)
    page.Visible = true
end

controller.UnselectedPage = function(page)
    page.Visible = false
end
```

**Result**: Clicking HomeTab shows HomePage and hides SettingsPage. Clicking SettingsTab does the opposite.

### Example 2: Multi-Group Dashboard

**Studio Setup:**
```
DashboardGui
├── MainTabs
│   ├── OverviewTab (Tag: "Tab", PageGroup: "Main", PageId: "Overview")
│   └── AnalyticsTab (Tag: "Tab", PageGroup: "Main", PageId: "Analytics") 
├── SettingsTabs  
│   ├── GeneralTab (Tag: "Tab", PageGroup: "Settings", PageId: "General")
│   └── AdvancedTab (Tag: "Tab", PageGroup: "Settings", PageId: "Advanced")
└── Pages
    ├── OverviewPage (Tag: "Page", PageGroup: "Main", PageId: "Overview")
    ├── AnalyticsPage (Tag: "Page", PageGroup: "Main", PageId: "Analytics")
    ├── GeneralPage (Tag: "Page", PageGroup: "Settings", PageId: "General")
    └── AdvancedPage (Tag: "Page", PageGroup: "Settings", PageId: "Advanced")
```

**Script:**
```lua
local xenterface = require(game.ReplicatedStorage.xenterface)

-- Set up Main navigation
local mainController = xenterface:GetController("Main")
mainController.SelectedPage = function(page)
    page.Visible = true
    page.GroupTransparency = 0
end
mainController.UnselectedPage = function(page)
    page.Visible = false
end

-- Set up Settings navigation (independent of Main)
local settingsController = xenterface:GetController("Settings")
settingsController.SelectedTab = function(tab)
    tab.BackgroundColor3 = Color3.fromRGB(34, 139, 34) -- Green theme for settings
end
settingsController.UnselectedTab = function(tab)
    tab.BackgroundColor3 = Color3.fromRGB(128, 128, 128)
end
```

**Result**: Two independent navigation systems. Main tabs control main pages, Settings tabs control settings pages. They don't interfere with each other.

---

## xenterface API Reference

### Core Module

#### `xenterface:GetController(pageGroup: string): Controller`
Gets or creates a controller for the specified page group.

**Parameters:**
- `pageGroup`: The name of the page group to manage

**Returns:** Controller object for the specified group

**Example:**
```lua
local controller = xenterface:GetController("MainMenu")
```

#### `xenterface:FireController(pageGroup: string, pageId: string | number, rawTab: GuiButton?)`
Manually triggers a page/tab switch without requiring user interaction.

**Parameters:**
- `pageGroup`: The page group to operate on
- `pageId`: The page ID to switch to  
- `rawTab`: (Optional) The tab that triggered this switch

**Example:**
```lua
xenterface:FireController("MainMenu", "Settings") -- Switch to Settings page
```

#### `xenterface:GetElementById(elementId: string): GuiObject?`
Finds an element by its ElementId attribute.

**Parameters:**
- `elementId`: The unique identifier to search for

**Returns:** The first GuiObject found with that ElementId, or nil

#### `xenterface:GetElementsByClass(elementClass: string): {GuiObject}`
Finds all elements with the specified ElementClass attribute.

**Parameters:**
- `elementClass`: The class name to search for

**Returns:** Array of GuiObjects with that class

#### `xenterface:HasGroup(pageGroup: string): boolean`
Checks if a page group exists and has been initialized.

#### `xenterface:GetGroups(): {string}`
Returns an array of all active page groups.

#### `xenterface:GetHover(hover: GuiObject): Hover?`
Gets the Hover object associated with a GuiObject that has the "Hover" tag.

**Parameters:**
- `hover`: The GuiObject to get the Hover instance for

**Returns:** Hover object if the GuiObject is tagged with "Hover", nil otherwise

**Example:**
```lua
local hover = xenterface:GetHover(myButton)
if hover then
    hover.EnterFunction = function(obj)
        obj.BackgroundColor3 = Color3.new(0, 1, 0)
    end
    hover.LeaveFunction = function(obj)
        obj.BackgroundColor3 = Color3.new(1, 1, 1)
    end
end
```

### Controller Object

Controllers are returned by `xenterface:GetController()` and define callback functions for state changes.

#### Properties

**`controller.SelectedTab: (tab: GuiButton) -> nil`**
Called when a tab becomes selected. Define this function to add visual feedback or custom logic.

**`controller.UnselectedTab: (tab: GuiButton) -> nil`**
Called when a tab becomes unselected.

**`controller.SelectedPage: (page: GuiObject) -> nil`**
Called when a page becomes active/visible.

**`controller.UnselectedPage: (page: GuiObject) -> nil`**
Called when a page becomes inactive/hidden.

**Example:**
```lua
local controller = xenterface:GetController("MyGroup")

controller.SelectedTab = function(tab)
    print("Tab selected:", tab.Name)
    tab.BackgroundColor3 = Color3.new(0, 1, 0)
end

controller.SelectedPage = function(page)
    page.Visible = true
    page.Position = UDim2.new(0, 0, 0, 0) -- Slide in from left
end
```

### Hover Object

Hover objects are created automatically when a GuiObject is tagged with "Hover". They provide hooks for custom behavior during mouse interactions.

#### Properties

**`hover.Hover: GuiObject`**
Reference to the GuiObject this Hover instance controls.

**`hover.EnterFunction: (GuiObject) -> nil`**
Optional callback function that runs when the mouse enters the GuiObject. Called after any xenimation animations play.

**`hover.LeaveFunction: (GuiObject) -> nil`**
Optional callback function that runs when the mouse leaves the GuiObject. Called after any xenimation animations play.

#### Methods

**`hover:Destroy()`**
Cleans up the hover object and disconnects all mouse event connections. Called automatically when the GuiObject is destroyed.

**Example:**
```lua
local hover = xenterface:GetHover(myButton)
if hover then
    hover.EnterFunction = function(guiObj)
        print("Hovered over:", guiObj.Name)
        guiObj.BorderSizePixel = 3
    end
    
    hover.LeaveFunction = function(guiObj)
        print("Stopped hovering over:", guiObj.Name)  
        guiObj.BorderSizePixel = 0
    end
    
    -- Later, if needed:
    hover:Destroy()
end
```

### Tags

Tags are applied to GuiObjects in Roblox Studio using the CollectionService tag editor.

#### `Tab`
Marks a GuiObject as a clickable tab that can switch pages.

**Required Attributes:**
- `PageGroup` (string): Which group this tab belongs to
- `PageId` (string): Which page this tab should activate

**Optional Attributes:**
- `TabExclusive` (boolean): If true, this tab won't be affected by other tabs in the same group

#### `Page` 
Marks a GuiObject as a page that can be shown/hidden by tabs.

**Required Attributes:**
- `PageGroup` (string): Which group this page belongs to  
- `PageId` (string): The unique identifier for this page

#### `Hover`
Enables automatic hover effects on the tagged GuiObject. When tagged, the framework automatically creates a Hover object that can be retrieved with `GetHover()`.

**Automatic Behavior:**
- Plays animations on mouse enter/leave (if xenimation animations are configured)
- Provides hooks for custom hover behavior

**Usage:**
```lua
-- GuiObject must be tagged with "Hover" in Studio first
local hover = xenterface:GetHover(someGuiObject)
if hover then
    hover.EnterFunction = function(obj) 
        -- Custom hover enter behavior (called AFTER animations)
        obj.BorderSizePixel = 2
    end
    hover.LeaveFunction = function(obj)
        -- Custom hover leave behavior (called AFTER animations)
        obj.BorderSizePixel = 0
    end
end
```

#### `Element`
Generic tag for finding UI elements by ID or class.

**Attributes:**
- `ElementId` (string): Unique identifier for this element
- `ElementClass` (string): Class name for grouping similar elements

### Attributes Reference

#### Core Navigation Attributes

**`PageGroup` (string)**
Groups related tabs and pages together. Tabs and pages with the same PageGroup will interact with each other.

**`PageId` (string)**  
Unique identifier within a PageGroup. Tabs and pages with matching PageId values are linked - clicking the tab will show the page.

#### Behavior Modifiers

**`TabExclusive` (boolean)**
When set to true on a tab, that tab will not be affected by other tabs in the same group. Useful for special tabs that should maintain their state.

**`ElementId` (string)**
Unique identifier for finding specific elements via `GetElementById()`.

**`ElementClass` (string)**
Class name for grouping elements, retrievable via `GetElementsByClass()`.
# xenimation Documentation

## Overview

**xenimation** is a utility-class based animation system inspired by Tailwind CSS, designed specifically for Roblox UI. It transforms complex TweenService code into simple, declarative class strings that describe animations through intuitive syntax.

### Key Principles

**Utility-First Approach**: Instead of writing custom animation functions, you compose behaviors using small, reusable utility classes like `scale-1.05`, `bgc-blue-0.8`, and `t-0.3`.

**State-Based Animation**: Every animation has two states - **Active** and **Inactive** - allowing for smooth transitions between UI states without manual state tracking.

**Declarative Configuration**: Animations are defined through attributes and presets, not code. This makes animations highly reusable and easy to modify without touching scripts.

**Automatic Property Capture**: The `initial` keyword automatically captures starting values, enabling smooth reversible animations without hardcoding default values.

**Seamless xenterface Integration**: While xenimation can be used independently, it truly shines when paired with xenterface's navigation system. This combination eliminates the need to manually trigger animations - they happen automatically when tabs are selected, pages change, or elements are hovered. You'll see how this integration makes complex UI animations completely effortless in the sections below.

### Traditional vs xenimation

**Traditional TweenService:**
```lua
-- Complex, repetitive code for each animation state
local function playSelectedAnimation(button)
    local selectedTween = TweenService:Create(button, 
        TweenInfo.new(0.3, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
        Size = UDim2.new(0, button.Size.X.Offset * 1.05, 0, button.Size.Y.Offset * 1.05),
        BackgroundColor3 = Color3.fromRGB(88, 101, 242),
        TextColor3 = Color3.fromRGB(255, 255, 255)
    })
    selectedTween:Play()
end

local function playUnselectedAnimation(button)
    -- Need to track/hardcode original values
    local unselectedTween = TweenService:Create(button,
        TweenInfo.new(0.1, Enum.EasingStyle.Linear, Enum.EasingDirection.Out), {
        Size = UDim2.new(0, 200, 0, 40), -- Hardcoded original size
        BackgroundColor3 = Color3.fromRGB(64, 64, 64), -- Hardcoded original color  
        TextColor3 = Color3.fromRGB(200, 200, 200) -- Hardcoded original text color
    })
    unselectedTween:Play()
end

-- playSelectedAnimation(button)   -- When selected
-- playUnselectedAnimation(button) -- When unselected
```

**xenimation Approach:**
```lua
-- Simple, readable utility classes - no hardcoding needed
-- Active = "scale-1.05 bgc-blue-0.8 tc-white-1 t-0.3 back out"
-- Inactive = "initial t-0.1 linear"

xenimation:Play(button, "default", true)  -- Plays Active state (selected)
xenimation:Play(button, "default", false) -- Plays Inactive state (unselected)
```

---

## xenimation Setup

### Step 1: Verify xenterface Integration
xenimation is built into xenterface, so if you have xenterface installed, you already have xenimation:

```lua
local xenterface = require(game.ReplicatedStorage.xenterface)
local xenimation = xenterface.xenimation -- Access xenimation directly
```

### Step 2: Tag elements for animation
Add the `Animation` tag to any GuiObject you want to animate in Roblox Studio.

### Step 3: Configure animation states
Add attributes to define your animation:

**Basic Setup:**
- `Active` (string): Utility classes for the "on" or "selected" state
- `Inactive` (string): Utility classes for the "off" or "unselected" state

**Example Attributes:**
- Active = `"scale-1.1 bgc-red-0.8 t-0.2 back out"`
- Inactive = `"initial bgc-gray-0.3 t-0.1 linear"`

### Step 4: Trigger animations
```lua
-- Manual triggering (when used independently)
xenimation:Play(myButton, "default", true)  -- Play Active state
xenimation:Play(myButton, "default", false) -- Play Inactive state

-- Or let xenterface handle it automatically (see Integration sections below)
-- This is where xenimation becomes incredibly powerful and effortless!
```

> **💡 Coming Up: Effortless Integration**
> 
> While the manual approach above works great, xenimation's real power emerges when integrated with xenterface. In the **Integration with xenterface Navigation** section, you'll see how animations can trigger automatically without any `xenimation:Play()` calls - just by clicking tabs or hovering over elements. This makes complex animated UIs incredibly simple to build.

---

## Multiple Animations per GuiObject

Like xenterface tabs/pages, you can add multiple animation configurations to a single GuiObject using **Configuration objects** as children.

### Setup Example:
```
MyButton (GuiButton - Tag: "Animation")
├── HoverAnimation (Configuration - Tag: "Animation")
│   ├── AnimationId = "hover"
│   ├── Active = "scale-1.05 t-0.2 back out"
│   └── Inactive = "initial t-0.1 linear"
└── ClickAnimation (Configuration - Tag: "Animation")
    ├── AnimationId = "click" 
    ├── Active = "scale-0.95 t-0.1 linear"
    └── Inactive = "initial t-0.1 linear"
```

### Usage:
```lua
-- Play different animations on the same object
xenimation:Play(MyButton, "default", true)  -- Plays button's main animation
xenimation:Play(MyButton, "hover", true)    -- Plays hover effect
xenimation:Play(MyButton, "click", true)    -- Plays click effect
```

**Key Points:**
- The main GuiObject can have a "default" animation (no AnimationId needed)
- Configuration children need an `AnimationId` attribute to distinguish them
- Each animation is independent and can play simultaneously
- Multiple animations on the same properties will override each other

---

## Animation Presets

Presets allow you to define reusable animation patterns in the `xenimation/Presets` module, promoting consistency across your UI.

### Defining Presets

Edit the `Presets` module inside the xenimation framework:

```lua
return {
    hover = {
        Active = "scale-1.05 t-0.2 back out",
        Inactive = "initial t-0.1 linear"
    },
    
    buttonPress = {
        Active = "scale-0.95 t-0.1 linear", 
        Inactive = "initial t-0.1 linear"
    },
    
    tabSelected = {
        Active = "scale-1.05 bgc-blue-0.8 tc-white-1 t-0.3 back out",
        Inactive = "initial bgc-gray-0.3 tc-gray-0.8 t-0.2 linear"
    },
    
    slideInLeft = {
        Active = "pos-c t-0.4 back out",
        Inactive = "pos-w--200xo gt-1 t-0.3 quad out"
    }
}
```

### Using Presets

**Method 1: Preset Attribute**
Set the `Preset` attribute on your GuiObject or Configuration:
- Preset = `"hover"` (single preset)
- Preset = `"hover buttonPress"` (multiple presets - they combine)

**Method 2: Explicit Override**
Use presets as a base and override specific values:
```lua
-- In attributes:
-- Preset = "hover"
-- Active = "scale-1.1 bgc-green-0.8" -- Overrides preset's Active
-- (Inactive will still use preset's value)
```

### Preset Priority
When combining presets with explicit values:
1. **Presets** are applied first (in order if multiple)
2. **Explicit Active/Inactive** override preset values
3. **Later presets** override earlier ones for conflicting properties

---

## Manual Animation Control

You can use xenimation independently of xenterface for full programmatic control:

### Example: Interactive Button with Multiple States

```lua
local xenterface = require(game.ReplicatedStorage.xenterface)
local xenimation = xenterface.xenimation

-- Get your button (assuming it's tagged with "Animation")
local button = player.PlayerGui.MainGui.ActionButton

-- Set up hover effects
button.MouseEnter:Connect(function()
    xenimation:Play(button, "hover", true)
end)

button.MouseLeave:Connect(function()
    xenimation:Play(button, "hover", false)
end)

-- Set up click animation
button.MouseButton1Down:Connect(function()
    xenimation:Play(button, "click", true)
end)

button.MouseButton1Up:Connect(function()
    xenimation:Play(button, "click", false)
end)

-- Set up selection state (e.g., for toggle buttons)
local isSelected = false
button.MouseButton1Click:Connect(function()
    isSelected = not isSelected
    xenimation:Play(button, "selected", isSelected)
end)

-- You can also check what animations are available
local availableAnimations = xenimation:GetAnimations(button)
print("Available animations:", table.concat(availableAnimations, ", "))

-- Or check if a specific animation exists
if xenimation:HasAnimation(button, "special") then
    xenimation:Play(button, "special", true)
end
```

**Required Studio Setup:**
```
ActionButton (GuiButton - Tag: "Animation")
├── HoverAnimation (Configuration - Tag: "Animation", AnimationId: "hover")
├── ClickAnimation (Configuration - Tag: "Animation", AnimationId: "click")  
└── SelectedAnimation (Configuration - Tag: "Animation", AnimationId: "selected")
```

---

# Integration with xenterface Navigation

xenimation integrates seamlessly with xenterface's navigation system (Tab, Page, Hover) through automatic animation triggering.

## Tab and Page Integration

### Using the Animations Attribute

The **Animations** attribute is used on **navigation elements**—anything with a **hover, page, or tab**.

- **Purpose:** Specifies which animations to play when the element is hovered, clicked, or activated.  
- **Multiple Animations:** Play more than one animation at once by listing their IDs in a **space-separated string**.  
- **Example:** If you have two animations, `hover1` and `hover2`, the attribute string would be "hover1 hover2"

xenterface will automatically play all listed animations when the element triggers.

```
HomeTab (GuiButton)
├── Tags: "Tab"
├── PageGroup: "MainMenu"
├── PageId: "Home"
├── Animations: "selected"
└── SelectedAnim (Configuration - Tag: "Animation", AnimationId: "selected")
    ├── Active: "scale-1.05 bgc-blue-0.8"
    └── Inactive: "initial bgc-gray-0.3"
```

**What happens:**
- When tab is selected: "selected" animation plays in Active state
- When tab is unselected: "selected" animation plays in Inactive state

### Automatic Default Animation (with Presets)

```
SimpleTab (GuiButton)
├── Tags: "Tab", "Animation"
├── PageGroup: "MainMenu"  
├── PageId: "Settings"
└── Preset: "tabSelected"
```

Result: Uses the preset's animation data automatically when tab is clicked.

## Hover Integration

### Simple Hover (with Presets)

```
HoverButton (GuiButton)
├── Tags: "Hover", "Animation"
└── Preset: "hover"
```

**What happens:**
- Mouse enter: Preset's Active state plays
- Mouse leave: Preset's Inactive state plays

### Multiple Hover Animations

```
HoverButton (GuiButton)
├── Tags: "Hover"
├── Animations: "glow pulse"
├── GlowAnim (Configuration - Tag: "Animation", AnimationId: "glow")
│   └── Preset: "buttonGlow"
└── PulseAnim (Configuration - Tag: "Animation", AnimationId: "pulse")
    ├── Active: "scale-mult-1.1 t-0.5 sine inOut"
    └── Inactive: "initial t-0.5 sine inOut"
```

## Advanced: Configuration References Multiple Animations

```
ComplexButton (GuiButton - Tag: "Animation", AnimationId: "self")
├── Active: "bgc-green-0.8"
├── Inactive: "initial"
├── ScaleAnim (Configuration - Tag: "Animation", AnimationId: "scale") 
│   ├── Active: "scale-1.1"
│   └── Inactive: "initial"
└── HoverConfig (Configuration - Tag: "Hover")
    └── Animations: "self scale"
```

**What happens:**
- Mouse enter: Plays both "self" (green background) AND "scale" (1.1x size) animations
- Mouse leave: Both animations return to their initial states
- No circular dependencies - animations call animations, not navigation

## Combined Tab + Hover

```
InteractiveTab (GuiButton)
├── Tags: "Tab", "Hover", "Animation"
├── PageGroup: "MainMenu"
├── PageId: "Home"  
├── Active: "scale-1.05 bgc-blue-0.8"     ← Tab selection
├── Inactive: "initial bgc-gray-0.3"
└── HoverEffect (Configuration - Tag: "Animation", AnimationId: "hover")
    ├── Active: "bgt-0.1"                  ← Hover effect (different property)
    └── Inactive: "initial"
```

**Behavior:**
- Hovering: Reduces background transparency (bgt-0.1)
- Clicking: Scales up and changes color
- Both can be active simultaneously since they affect different properties

## Property Conflicts and Edge Cases

### Conflicting Properties

When multiple animations modify the same property, the last one applied takes precedence:

```
ConflictingExample
├── Tags: "Tab", "Hover", "Animation"
├── Active: "scale-1.05"                   ← Tab: scale to 1.05
└── HoverConfig (Tag: "Animation", AnimationId: "hover")
    ├── Active: "scale-1.1"                ← Hover: scale to 1.1 (overwrites)
    └── Inactive: "initial"
```

If you hover first, then click, the final scale will be 1.05 (tab wins). If you click then hover, the final scale will be 1.1 (hover wins).

### Initial Keyword Issues

Be careful when using `initial` with multiple animation systems:

```
ProblematicExample
├── Active: "scale-1.2 bgc-red-0.8"       ← Changes scale AND color
└── HoverConfig Active: "initial bgc-blue-0.8"  ← Resets scale, changes color
```

When hovering, this will reset the scale to original value even if the tab is selected, which may not be desired. Better approach:

```
BetterExample  
├── Active: "scale-1.2 bgc-red-0.8"
└── HoverConfig Active: "bgc-blue-0.8"    ← Only changes color, preserves scale
```

---

## Quick Animation Shortcuts

For simple animations, you can skip creating Configuration objects and put animation classes directly on the main GuiObject:

### Method 1: Direct Attributes (No AnimationId)
```
QuickButton (GuiButton - Tag: "Animation")
├── Active: "scale-1.1 bgc-green-0.8 t-0.2 back"
└── Inactive: "initial t-0.1 linear"
```

### Method 2: Direct Preset (No AnimationId)  
```
QuickButton (GuiButton - Tag: "Animation")
└── Preset: "hover"
```

### Method 3: Mixed Approach
```  
QuickButton (GuiButton - Tag: "Animation")
├── Preset: "hover" 
└── Active: "bgc-purple-0.8" ← Overrides preset's background color
```

**All three methods** create a "default" animation that can be triggered with:
```lua
xenimation:Play(QuickButton, "default", true)  -- Or just:
xenimation:Play(QuickButton, nil, true)        -- nil defaults to "default"
```

This is perfect for simple hover effects, button presses, or any single-state animation where you don't need multiple animation types.

---

## Advanced Features

### The `initial` Keyword

The `initial` keyword is one of xenimation's most powerful features. It automatically captures and restores the starting values of any properties you animate, eliminating the need to hardcode default values.

**How it works:**
1. When an element is tagged with "Animation", xenimation automatically captures its current property values
2. The `initial` keyword in any animation state restores those captured values
3. This enables smooth reversible animations without knowing the original values

**Example Problem (Traditional Approach):**
```lua
-- Without initial - you need to know and hardcode the starting values
local button = someButton
-- What if the button's original scale isn't exactly 1? What if it's positioned differently?
-- You'd need to manually check and hardcode these values
local originalScale = button.UIScale and button.UIScale.Scale or 1
local originalColor = button.BackgroundColor3
-- ... etc for every property
```

**xenimation Solution:**
```
MyButton (GuiButton - Tag: "Animation")
├── Active: "scale-1.2 bgc-green-0.8 rot-10"
└── Inactive: "initial" ← Automatically restores original scale, color, AND rotation
```

**Real-world Example:**
```
-- Your designer gives you a button that's already styled:
-- Scale = 0.95, BackgroundColor3 = RGB(120, 120, 120), Rotation = -2

StyledButton (GuiButton - Tag: "Animation") 
├── Active: "scale-1.1 bgc-blue-0.8 rot-5"
└── Inactive: "initial t-0.2 quad out"
```

**Result:**
- **Active state**: Scale becomes 1.1, color becomes blue, rotation becomes 5°
- **Inactive state**: Scale returns to 0.95, color returns to RGB(120,120,120), rotation returns to -2°
- **No hardcoding required** - works regardless of the button's starting values

### Relative Operations

Relative operations let you modify existing values instead of setting absolute ones. This is incredibly useful for animations that need to adapt to different starting states.

**Operation Types:**
- `add`: Addition (`pos-add-50xo` = "add 50 pixels to X offset")
- `sub`: Subtraction (`scale-sub-0.1` = "subtract 0.1 from current scale") 
- `mult`: Multiplication (`pos-mult-1.5` = "multiply position by 1.5")
- `div`: Division (`rot-div-2` = "divide rotation by 2")

**Why Use Relative Operations:**

**Problem**: You want a "shake" effect that moves any button 10 pixels right, regardless of its current position.

**Absolute Approach (Doesn't Work):**
```
-- This only works if the button is at exactly pos-c
ShakeButton Active: "pos-c-10xo"    ← Hardcoded center + 10 pixels
ShakeButton Inactive: "pos-c"       ← Hardcoded center
```

**Relative Approach (Works Everywhere):**
```
ShakeButton Active: "pos-add-10xo"   ← Add 10 pixels to current X offset  
ShakeButton Inactive: "initial"     ← Return to wherever it started
```

**Complex Example - Pulse Effect:**
```
PulseButton (GuiButton - Tag: "Animation")
├── Active: "scale-mult-1.1 bgt-sub-0.2 t-0.6 sine inOut"
└── Inactive: "initial t-0.6 sine inOut"
```

**What this does:**
- **Active**: Multiplies current scale by 1.1 (10% bigger) AND reduces transparency by 0.2 (more visible)
- **Inactive**: Returns to original scale and transparency
- **Works on any button** regardless of starting scale (0.5, 1.0, 1.3, etc.) or transparency

**Circular Dependency Protection:**
```
-- ❌ This creates a circular dependency (both states use relative operations on same property)
BadButton Active: "scale-add-0.1"
BadButton Inactive: "scale-sub-0.1"  ← Can't determine base value!

-- ✅ This works (only one state uses relative operations)
GoodButton Active: "scale-add-0.1"
GoodButton Inactive: "initial"       ← Uses absolute captured value as base
```

xenimation automatically detects and warns about circular dependencies to prevent animation issues.

---

## Utility Class Reference

### Position & Layout

| Class Pattern | Property | Description | Example |
|---------------|----------|-------------|---------|
| `pos-{position}` | Position | Absolute positioning with cardinal directions and offsets | `pos-c` (center), `pos-nw-50xo-25yo` (northwest + offset) |
| `ap-{vector2}` | AnchorPoint | Anchor point for positioning | `ap-0.5-0.5` (center anchor) |
| `cp-{vector2}` | CanvasPosition | Canvas position for ScrollingFrames | `cp-100-50` |
| `rot-{degrees}` | Rotation | Rotation in degrees | `rot-45`, `rot-cw-90`, `rot-ccw-30` |

### Colors (RGB-based with Intensity)

| Class Pattern | Property | Description | Example |
|---------------|----------|-------------|---------|
| `bgc-{color}-{intensity}` | BackgroundColor3 | Background color with intensity (0.0-1.0) | `bgc-blue-0.8`, `bgc-red-0.5` |
| `tc-{color}-{intensity}` | TextColor3 | Text color | `tc-white-1`, `tc-gray-0.6` |
| `ic-{color}-{intensity}` | ImageColor3 | Image tint color | `ic-green-0.7` |
| `tsc-{color}-{intensity}` | TextStrokeColor3 | Text outline color | `tsc-black-0.8` |
| `bc-{color}-{intensity}` | BorderColor3 | Border color | `bc-blue-0.9` |
| `gc-{color}-{intensity}` | GroupColor3 | Group color | `gc-red-0.4` |

**Available Colors:**
`red`, `orange`, `yellow`, `green`, `blue`, `indigo`, `violet`, `purple`, `pink`, `brown`, `tan`, `beige`, `cyan`, `magenta`, `lime`, `teal`, `navy`, `maroon`, `olive`, `black`, `gray`, `white`

### Transparency

| Class Pattern | Property | Description | Example |
|---------------|----------|-------------|---------|
| `bgt-{value}` | BackgroundTransparency | Background transparency (0-1) | `bgt-0.5` |
| `tt-{value}` | TextTransparency | Text transparency | `tt-0.2` |
| `it-{value}` | ImageTransparency | Image transparency | `it-0.8` |
| `gt-{value}` | GroupTransparency | Group transparency (affects all descendants) | `gt-1` (invisible) |

### UI Modifiers

| Class Pattern | Target | Property | Description | Example |
|---------------|---------|----------|-------------|---------|
| `scale-{value}` | UIScale | Scale | Scale multiplier | `scale-1.05`, `scale-0.9` |
| `sth-{value}` | UIStroke | Thickness | Stroke thickness in pixels | `sth-2` |
| `str-{value}` | UIStroke | Transparency | Stroke transparency | `str-0.3` |
| `sc-{color}-{intensity}` | UIStroke | Color | Stroke color | `sc-blue-0.8` |
| `cr-{udim}` | UICorner | CornerRadius | Corner radius | `cr-0-8` (8 pixels), `cr-0.1-0` (10% scale) |
| `ar-{value}` | UIAspectRatioConstraint | AspectRatio | Width/height ratio | `ar-1.77` (16:9) |

### Size Constraints

| Class Pattern | Target | Property | Description | Example |
|---------------|---------|----------|-------------|---------|
| `maxs-{vector2}` | UISizeConstraint | MaxSize | Maximum size | `maxs-500-300` |
| `mins-{vector2}` | UISizeConstraint | MinSize | Minimum size | `mins-100-50` |
| `maxts-{value}` | UITextSizeConstraint | MaxTextSize | Maximum text size | `maxts-24` |
| `mints-{value}` | UITextSizeConstraint | MinTextSize | Minimum text size | `mints-8` |

### Other Properties

| Class Pattern | Property | Description | Example |
|---------------|----------|-------------|---------|
| `tst-{value}` | TextStrokeThickness | Text outline thickness | `tst-2` |
| `bsp-{value}` | BorderSizePixel | Border size in pixels | `bsp-1` |

### Animation Timing

| Class Pattern | Property | Description | Example |
|---------------|----------|-------------|---------|
| `t-{duration}` | Duration | Animation duration in seconds | `t-0.3`, `t-1.5` |
| `t-xs` | Duration | Extra small (0.1s) | |
| `t-sm` | Duration | Small (0.2s) | |
| `t-md` | Duration | Medium (0.3s) | |
| `t-lg` | Duration | Large (0.5s) | |
| `t-xl` | Duration | Extra large (0.8s) | |

### Easing Styles

| Class | Easing Style | Class | Easing Style |
|--------|-------------|--------|-------------|
| `linear` | Linear | `sine` | Sine |
| `quad` | Quad | `exponential` | Exponential |
| `cubic` | Cubic | `circular` | Circular |
| `quart` | Quart | `back` | Back |
| `bounce` | Bounce | `elastic` | Elastic |

### Easing Directions

| Class | Direction | Description |
|--------|-----------|-------------|
| `in` | In | Start slow, end fast |
| `out` | Out | Start fast, end slow |
| `inOut` | InOut | Start slow, fast middle, end slow |

### Special Keywords

| Class | Description | Example |
|--------|-------------|---------|
| `initial` | Restores captured starting values | `"scale-1.1 bgc-blue-0.8"` → `"initial"` |

### Relative Operations

Add mathematical operations to existing values:

| Pattern | Operation | Example | Result |
|---------|-----------|---------|---------|
| `{property}-add-{value}` | Addition | `pos-add-50xo` | Adds 50 pixels to X offset |
| `{property}-sub-{value}` | Subtraction | `scale-sub-0.1` | Subtracts 0.1 from current scale |
| `{property}-mult-{value}` | Multiplication | `pos-mult-2` | Doubles current position values |
| `{property}-div-{value}` | Division | `rot-div-2` | Halves current rotation |

### Position Format Examples

| Format | Description | Example |
|--------|-------------|---------|
| Cardinal | Predefined positions | `pos-c` (center), `pos-nw` (northwest) |
| Cardinal + Offset | Cardinal with pixel offsets | `pos-c-100xo-50yo` (center + offset) |
| Direct UDim2 | Scale-Offset-Scale-Offset | `pos-0.5-100-0.3-50` |
| Scale Only | For relative positioning | `pos-0.25-0-0.75-0` |

**Cardinal Directions:**
- `nw` (northwest), `n` (north), `ne` (northeast)
- `w` (west), `c` (center), `e` (east)  
- `sw` (southwest), `s` (south), `se` (southeast)
