---
---

# Runtime Asset Import

[Back to plugin list](../)

## Overview

Runtime Asset Import is an Unreal Engine code plugin that imports static 3D mesh data at runtime on Windows (Win64). It accepts a local file path or an in-memory byte array and constructs a hierarchy of Dynamic Mesh Components or Procedural Mesh Components.

Dynamic Mesh Components are the recommended output.

## Requirements

- Unreal Engine 5.4, 5.5, 5.7, or 5.8
- Windows Win64 editor or packaged application
- A static 3D mesh file accessible to the application, or equivalent in-memory data
- A parent material containing the required parameters described below

The bundled Win64 Assimp library is included in the Fab package. CMake and a separate Assimp installation are not required for normal use.

## Installation

### From Fab

1. Acquire **Runtime Asset Import** from Fab.
2. Install the plugin for a supported Unreal Engine version through the Epic Games Launcher.
3. Open the Unreal Engine project.
4. Open **Edit > Plugins**.
5. Enable **Runtime Asset Import** if it is not already enabled.
6. Restart the editor when requested.

### From Source

Place the repository at:

```text
<Project>/Plugins/RuntimeAssetImport
```

Then generate the project files or open the `.uproject` file and accept the rebuild prompt.

```powershell
git clone https://github.com/metyatech/RuntimeAssetImportPlugin.git Plugins/RuntimeAssetImport
```

Build the project for Win64.

## Quick Start

### Blueprint

1. Prepare a local FBX, OBJ, glTF, GLB, or DAE file.
2. Add **Construct Dynamic Mesh Component from Asset File** to a Blueprint.
3. Pass the local file path.
4. Pass an owning Actor.
5. Pass a parent material that contains the required parameters.
6. Handle the `Success` and `Failure` execution outputs.
7. Use the returned root Dynamic Mesh Component. Imported child nodes are attached below it.

For in-memory input:

1. Call **Load Mesh from Asset Data**.
2. Pass the returned mesh data to **Construct Dynamic Mesh Component from Mesh Data**.

### C++

```cpp
#include "AssetConstructor.h"

EConstructDynamicMeshComponentFromAssetFileResult Result =
    EConstructDynamicMeshComponentFromAssetFileResult::Failure;

UDynamicMeshComponent* RootComponent =
    UAssetConstructor::ConstructDynamicMeshComponentFromAssetFile(
        FilePath,
        ParentMaterial,
        OwnerActor,
        Result);

if (Result != EConstructDynamicMeshComponentFromAssetFileResult::Success ||
    RootComponent == nullptr)
{
    // Handle the failed import.
}
```

## Features

* Synchronous runtime import from a local file
* Synchronous runtime import from an in-memory byte array
* Blueprint and C++ APIs
* Dynamic Mesh Component hierarchy construction
* Procedural Mesh Component hierarchy construction
* Hierarchical node transforms
* Multiple mesh sections
* Imported diffuse or base color
* One embedded texture per material
* Collision generation on constructed components
* FBX, OBJ, glTF, GLB, and DAE importers
* Assimp 6.0.5 built from the official source tag

## Settings / Blueprint Nodes

### Parent Material Parameters

The parent material must contain all three parameters.

| Parameter                           | Type      | Purpose                                      |
| ----------------------------------- | --------- | -------------------------------------------- |
| `TextureBlendIntensityForBaseColor` | Scalar    | Selects the base-color texture contribution. |
| `BaseColor4`                        | Vector    | Supplies the imported diffuse or base color. |
| `BaseColorTexture`                  | Texture2D | Supplies the imported embedded texture.      |

The bundled `/RuntimeAssetImport/AssetImporterMeshMaterial` material satisfies this contract.

A parent material missing any required parameter is rejected before a partial component hierarchy is created.

### Blueprint Functions

The plugin exposes these six Blueprint-callable functions:

* `LoadMeshFromAssetFile`
* `LoadMeshFromAssetData`
* `ConstructDynamicMeshComponentFromMeshData`
* `ConstructDynamicMeshComponentFromAssetFile`
* `ConstructProceduralMeshComponentFromMeshData`
* `ConstructProceduralMeshComponentFromAssetFile`

The construction functions include `ShouldRegisterComponentToOwner`. Leave it enabled for normal use unless the caller will register and manage the returned components manually.

## Limitations

* Win64 only
* Static geometry only
* No skeletal mesh import
* No animation import
* No morph target import
* No LOD import
* No latent or asynchronous API
* No URL downloader
* External texture references are not loaded
* Only the first material texture is used
* Only the first UV channel is used
* Only the first vertex-color channel is used
* Runtime-created meshes are local and are not automatically replicated
* Input paths must refer to local files accessible to the application
* Procedural Mesh Components may cause movement or network issues when independently created by multiplayer peers
* Large or complex imports block the calling thread until parsing and construction finish

Use Dynamic Mesh Components instead of Procedural Mesh Components unless the project specifically requires the procedural component path.

## Troubleshooting

### The failure execution output is used

Check all of the following:

* The file exists at the supplied local path.
* The packaged application has permission to read the file.
* The file contains supported static geometry.
* The file is FBX, OBJ, glTF, GLB, or DAE.
* The parent material contains all three required parameters.
* The plugin is running on Win64.

### The mesh appears without an external texture

External texture references are not loaded. Use an embedded texture or apply the required material and texture separately after import.

### Importing causes a visible pause

All import and construction APIs are synchronous. Validate file size before import and call the API at a point where blocking the calling thread is acceptable.

### Multiplayer movement or network behavior is incorrect

Use the Dynamic Mesh Component functions. The plugin does not replicate imported mesh data or created component hierarchies automatically.

For reproducible problems, open an issue in the [RuntimeAssetImportPlugin repository](https://github.com/metyatech/RuntimeAssetImportPlugin/issues). Include the Unreal Engine version, input format, editor or packaged-build status, and a minimal reproduction.

## Version History

### 1.0.0

* Initial Fab release
* Runtime file and memory import
* Dynamic Mesh and Procedural Mesh construction
* FBX, OBJ, glTF, GLB, and DAE support
* Win64 support
