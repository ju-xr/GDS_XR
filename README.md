Spatial Anchor Plugin for Unity (Meta Building Blocks) Complete Walkthrough Tutorial can be view from here. https://youtu.be/gxvjO3nHaeI
# Spatial Anchor Local Loader

A Unity package for managing and loading Meta XR Spatial Anchors with persistent storage and flexible prefab spawning.

## Overview

This project provides a robust solution for loading and managing Spatial Anchors in Unity applications targeting Meta Quest devices. It enables you to:
- Load spatial anchor UUIDs from local storage
- Spawn prefabs at anchor positions with flexible mapping options
- Persist anchor data in ScriptableObjects for version control
- Support both bound (fixed) and unbound (grabbable) spawning modes
- Manage anchor lifecycle including erasure

## Features

- **Persistent Storage**: Loads anchor UUIDs from Unity's `PlayerPrefs` (compatible with Meta XR Building Blocks)
- **Flexible Prefab Mapping**: Assign different prefabs to specific anchor UUIDs
- **Dual Spawning Modes**:
  - **Unbound Spawning**: Objects spawn at anchor locations but can be moved/grabbed
  - **Bound Spawning**: Objects are bound to anchors via SDK (fixed positions)
- **UUID Database Asset**: ScriptableObject-based storage for UUIDs and prefab mappings
- **Auto-sync**: Automatically syncs loaded UUIDs to database assets
- **Context Menu Operations**: Easy-to-use editor tools for importing and managing UUIDs
- **Keyboard Shortcuts**: Quick testing controls (R for refresh, S for spawn)

## Requirements

### Unity Version
- **Unity 6000.0.61f1** (or compatible Unity 6.x version)

### Required Packages
- **Meta XR SDK All-in-One** (`com.meta.xr.sdk.all`) version 78.0.0 or higher
- **Meta XR Building Blocks** (SpatialAnchorCoreBuildingBlock)
- Unity XR Management
- Unity OpenXR

### Platform Support
- Meta Quest 2/3/Pro
- Requires Meta XR SDK for Spatial Anchors functionality

## Installation

1. Clone this repository or download the project files
2. Open the project in Unity 6000.0.58f1 (or compatible version)
3. Ensure the Meta XR SDK is installed via Unity Package Manager
4. Import the Meta XR Building Blocks sample (if not already present)

## Project Structure

```
Assets/
├── Scripts/
│   ├── SpatialAnchorLocalLoader.cs      # Main loader component
│   └── SpatialAnchorUuidDatabase.cs     # UUID database ScriptableObject
└── ...
```

## Usage

### Basic Setup

1. **Add the Component**:
   - Add `SpatialAnchorLocalLoader` component to a GameObject in your scene
   - Ensure you have a `SpatialAnchorCoreBuildingBlock` in the scene

2. **Configure the Loader**:
   - Assign an `Anchor Prefab` in the Inspector (default prefab for all anchors)
   - Optionally set `Spawn Parent` to organize spawned objects in the hierarchy

3. **Runtime Behavior**:
   - Set `Load On Start` to automatically load UUIDs when the scene starts
   - Set `Spawn On Start` to automatically spawn objects after loading
   - Adjust `Spawn On Start Delay Seconds` if needed for pose tracking to stabilize

### Prefab Mapping

#### Method 1: Inspector Mapping
1. In the `SpatialAnchorLocalLoader` component, expand "Per-UUID Prefab Mappings"
2. Add entries with UUID strings and corresponding prefabs
3. At spawn time, the loader will match UUIDs and use the appropriate prefab

#### Method 2: UUID Database Asset
1. Create a `SpatialAnchorUuidDatabase` asset (Right-click → Create → SpatialAnchors → Uuid Database)
2. Assign it to the loader's `UUID Database Asset` field
3. Configure UUID-to-prefab mappings in the asset
4. Enable `Auto Save Loaded UUIDs To Asset` to automatically sync loaded UUIDs

### Spawning Modes

#### Unbound Spawning (Default)
- **Use Unbound Spawning**: `true`
- Objects spawn at anchor positions but are not bound to the anchor
- Allows objects to be grabbed and moved
- Requires `Ensure Grabbable Components` if you want physics-based interaction

#### Bound Spawning
- **Use Unbound Spawning**: `false`
- Objects are bound to anchors via the SDK
- Objects remain fixed at anchor positions
- Useful for persistent environmental elements

### Editor Tools (Context Menu)

Right-click the component in the Inspector to access:

- **Refresh UUIDs From Local Storage**: Load UUIDs from PlayerPrefs
- **Spawn Anchors From Prefab**: Manually trigger spawning
- **Import UUIDs Into Mappings**: Import all UUIDs from PlayerPrefs into the mappings list
- **Save Loaded UUIDs To Inspector**: Persist loaded UUIDs to the scene
- **Save Loaded UUIDs To Asset**: Save UUIDs to the database asset
- **Load UUIDs From Asset Into Inspector**: Load UUIDs from asset to component
- **Erase Anchors (device) From eraseOnStartUuids**: Remove anchors from device storage

### Keyboard Shortcuts (Play Mode)

- **R**: Refresh UUIDs from local storage
- **S**: Spawn anchors from prefabs
- (Shortcuts can be disabled by unchecking `Enable Keyboard Shortcuts`)

### Anchor Management

#### Erasing Anchors

1. **On Start Erase**: 
   - Enable `Erase On Start`
   - Add UUIDs to `Erase On Start Uuids` list
   - Anchors will be erased from device storage on scene start

2. **On Refresh Erase**:
   - Enable `Erase On Refresh`
   - UUIDs in `Erase On Start Uuids` will be removed on refresh

## Component Settings

### Main Settings
- **Anchor Prefab**: Default prefab to spawn at anchor locations
- **Spawn Parent**: Transform to parent spawned objects under
- **Load On Start**: Automatically load UUIDs on Start()
- **Spawn On Start**: Automatically spawn objects on Start()
- **Spawn On Start Delay Seconds**: Delay before spawning (allows tracking to stabilize)

### Spawning Options
- **Use Unbound Spawning**: Enable unbound (grabbable) or bound (fixed) spawning
- **Ensure Grabbable Components**: Auto-add Rigidbody and Collider to spawned objects
- **Spawned Use Gravity**: Gravity setting for spawned Rigidbodies
- **Spawned Is Kinematic**: Kinematic setting for spawned Rigidbodies

### UUID Management
- **UUID Database Asset**: Reference to ScriptableObject for persistent UUID storage
- **Auto Save Loaded UUIDs To Asset**: Automatically sync loaded UUIDs to asset

### Debug
- **Enable Keyboard Shortcuts**: Toggle keyboard shortcuts in play mode
- **Refresh Key**: Key to trigger refresh (default: R)
- **Spawn Key**: Key to trigger spawn (default: S)

## Integration with Meta XR Building Blocks

This loader is designed to work with the Meta XR Building Blocks system:
- Reads UUIDs from the same PlayerPrefs keys used by `SpatialAnchorLocalStorageManagerBuildingBlock`
- Can use `SpatialAnchorCoreBuildingBlock` for bound spawning mode
- Compatible with the standard Meta XR Spatial Anchor workflow

## Technical Details

### UUID Storage Format
UUIDs are stored in PlayerPrefs using the following format:
- `numUuids`: Integer count of stored UUIDs
- `uuid0`, `uuid1`, ...: String values for each UUID

### Spawning Process
1. UUIDs are loaded from PlayerPrefs or Inspector
2. UUIDs are filtered (exclusions applied if configured)
3. For unbound spawning:
   - Unbound anchors are loaded via `OVRSpatialAnchor.LoadUnboundAnchorsAsync()`
   - Each anchor is localized
   - A temporary bound anchor is created to resolve pose
   - Prefab is instantiated at resolved pose
   - Temporary anchor is destroyed
4. For bound spawning:
   - Uses `SpatialAnchorCoreBuildingBlock.LoadAndInstantiateAnchors()`

## Limitations

- Requires Meta Quest device or compatible Meta XR hardware
- Spatial Anchors require device-specific spatial tracking
- Unbound spawning creates temporary bound anchors for pose resolution (performance consideration)
- Erase operations require device storage access

## Troubleshooting

### No UUIDs Loaded
- Ensure anchors have been created and saved using Meta XR Building Blocks
- Check that `Load On Start` is enabled or manually call `Refresh UUIDs From Local Storage`
- Verify PlayerPrefs contains UUID entries

### Objects Not Spawning
- Verify `Anchor Prefab` or UUID mappings are assigned
- Check that UUIDs are valid GUID format
- Enable debug logging to see spawn process details
- Ensure sufficient delay for tracking system to stabilize

### Objects Not Grabbable
- Enable `Use Unbound Spawning`
- Enable `Ensure Grabbable Components`
- Verify your prefabs have appropriate Rigidbody and Collider components
- Check that your interaction system (e.g., OVRGrabber) is configured

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is open source. Please check the license file for details.

## Acknowledgments

- Built for Meta XR SDK and Unity
- Compatible with Meta XR Building Blocks
- Uses Unity's Spatial Anchor system via OVR (OpenXR)

## Support

For issues, questions, or contributions, please open an issue on the GitHub repository.

