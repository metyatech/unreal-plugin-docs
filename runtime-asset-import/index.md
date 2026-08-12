---
---

# Runtime Asset Import

[Back to plugin list](../)

## Overview

Runtime Asset Import is an Unreal Engine code plugin that imports static 3D mesh data at runtime on Windows (Win64). It accepts a local file path or an in-memory byte array and constructs a hierarchy of Dynamic Mesh Components or Procedural Mesh Components.

Dynamic Mesh Components are the recommended output.

**File imports resolve dependencies only inside the model directory sandbox. In-memory imports remain self-contained and never read external files.**

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
* First external or embedded diffuse/base-color texture per material
* File imports can resolve required auxiliary files such as OBJ material files and glTF external buffers
* File imports can load the first external diffuse/base-color texture
* Auxiliary files are sandboxed to the model directory and its subdirectories
* Embedded textures are supported by file and in-memory imports
* Collision generation on constructed components
* FBX, OBJ, glTF, GLB, and DAE importers
* Assimp 6.0.5 built from the official source tag

## Settings / Blueprint Nodes

### Parent Material Parameters

The parent material must contain all three parameters.

| Parameter | Type | Purpose |
| --- | --- | --- |
| `TextureBlendIntensityForBaseColor` | Scalar | Selects the base-color texture contribution. |
| `BaseColor4` | Vector | Supplies the imported diffuse or base color. |
| `BaseColorTexture` | Texture2D | Supplies the imported external or embedded diffuse/base-color texture. |

The bundled `/RuntimeAssetImport/RuntimeAssetImport/AssetImporterMeshMaterial` material satisfies this contract.

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

## Import Security and Resource Limits

### File Imports

File imports may resolve auxiliary files only when their final resolved target remains inside the model file's directory or a subdirectory.

This applies to:

* OBJ material files
* glTF external buffers
* Diffuse/base-color textures

References that escape through parent-directory traversal, outside-root absolute paths, junctions, symbolic links, device paths, URLs, alternate data streams, or auxiliary files with multiple hard links are rejected. **This prevents path references from escaping the selected model directory sandbox.**

For untrusted model packages, extract or copy the model and its intended dependencies into a dedicated directory before import. The sandbox prevents model references from resolving outside that directory. Files intentionally placed inside the directory are considered available to the model.

### In-Memory Imports

In-memory imports support self-contained data, data URIs, and embedded textures.

They do not read external material files, buffers, or textures because the current memory API does not receive a trusted base directory.

### Limits

Imports enforce these resource limits:

* Main model file: 512 MiB
* Individual auxiliary file: 256 MiB
* Compressed texture: 64 MiB
* Total unique files opened per import: 256
* Total unique opened bytes per import: 1 GiB
* Raw texture maximum dimension: 16384
* Raw texture maximum pixels: 67,108,864

File-count and total-byte budgets are charged when a file is actually opened, not when its existence is queried. If a file grows and is reopened during the same import, the additional bytes count toward the total budget. Shrinking a file does not refund budget during that import.

Compressed image headers are validated before texture decoding. Width, height, pixel count, and compressed byte limits apply to external textures, embedded textures, data URIs, and texture data supplied through mesh-data construction APIs.

## Limitations

* Win64 only
* Static geometry only
* No skeletal mesh import
* No animation import
* No morph target import
* No LOD import
* No latent or asynchronous API
* No URL downloader
* Only the first diffuse/base-color texture is used
* File imports resolve auxiliary files only inside the model directory sandbox
* In-memory imports do not resolve external files
* Import and texture size limits apply
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

Check all of the following:

* The texture path resolves inside the model directory or a subdirectory.
* The file exists and is readable.
* The texture is the first diffuse/base-color texture.
* The texture is within the documented size limit.
* In-memory imports require an embedded texture or data URI.

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
* Sandboxed external auxiliary-file and texture loading for file imports
* Embedded texture support for file and memory imports
* File, dependency, and texture resource limits
