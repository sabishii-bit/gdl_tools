# GDL Tools

A comprehensive suite of modding tools for **Gauntlet: Dark Legacy**, supporting PlayStation 2, GameCube, Xbox, and Arcade platforms. These tools allow you to extract, modify, and recompile game assets including 3D models, textures, animations, collision data, messages, and more.

## Table of Contents

- [Overview](#overview)
- [Installation](#installation)
- [Applications](#applications)
  - [Crucible](#crucible---asset-compilerdecompiler)
  - [GDL Archive Tool](#gdl-archive-tool---game-archive-manager)
  - [Legend Viewer](#legend-viewer---3d-asset-viewer)
- [Understanding the Workflow](#understanding-the-workflow)
- [File Structure](#file-structure)
- [Supported Formats](#supported-formats)
- [Platform Support](#platform-support)
- [License](#license)

## Overview

### What are these tools for?

Gauntlet: Dark Legacy stores its game data in a hierarchical structure:

1. **Game Archives (WAD.BIN / HDD)** - Container files that store all game data
   - PS2 uses `WAD.BIN` files
   - Arcade uses raw HDD images

2. **Cache Files (OBJECTS.PS2, WORLDS.PS2, *.ROM)** - Structured binary files inside archives
   - `OBJECTS.PS2/NGC` - Contains character, enemy, and item data
   - `WORLDS.PS2/NGC` - Contains level geometry and collision data
   - `*.ROM` files - Contains game text, fonts, enemy data, player stats, etc.

3. **Asset Data** - The actual 3D models, textures, and metadata buried in cache files
   - Models in proprietary G3D format
   - Textures in proprietary GTX format
   - Metadata in binary structures

### Tool Responsibilities

- **GDL Archive Tool** = Archive layer (WAD.BIN ↔ cache files)
- **Crucible** = Cache layer (cache files ↔ editable assets)
- **Legend Viewer** = Visualization (preview assets in 3D)

## Installation

### Prerequisites

- Python 3.9.13
- tkinter (usually included with Python)

### Dependencies

Install using:

```bash
pip install -r dependencies.txt
```

## Applications

### Crucible - Asset Compiler/Decompiler

**Purpose:** Crucible works with **cache files** (OBJECTS.PS2, WORLDS.PS2, *.ROM) to extract and compile the actual game assets (models, textures, text) into editable formats.

**What it does:**
- Extracts 3D models from binary cache files to OBJ format
- Extracts textures to PNG/TGA/DDS
- Extracts metadata (object properties, stats, etc.) to YAML/JSON
- Extracts game text and fonts from ROM files
- Compiles modified assets back into platform-specific cache files

**When to use it:**
- You want to edit 3D models, textures, or game data
- You have already extracted cache files from WAD.BIN (using GDL Archive Tool)
- You want to rebuild cache files after making modifications

#### Running Crucible

```bash
python Crucible.py
```

#### Interface Overview

Crucible has three main sections:

1. **Objects compilation** - For OBJECTS.PS2/NGC files (characters, enemies, items)
2. **Worlds compilation** - For WORLDS.PS2/NGC files (levels, collision) [Note: Currently decompile-only]
3. **Messages compilation** - For *.ROM files (text, fonts, game data)

#### Settings Explained

**Platform target:**
- Choose which console version you're modding
- PS2, GameCube, Xbox, or Arcade
- Affects texture formats, model optimizations, and file extensions

**File formats:**
- **Collision format**: OBJ (for extracting world collision meshes)
- **Model format**: Wavefront OBJ (industry standard, works with Blender/Maya)
- **Texture format**:
  - PNG - Lossless, good for editing
  - TGA - Preserves alpha channel perfectly
  - DDS - DirectX format with compression
- **Metadata format**:
  - YAML - Human-readable, easier to edit
  - JSON - Also readable, more standardized

**Performance options:**
- **Use parallel processing**: Splits work across CPU cores (2-4x faster)
- **Optimize models and textures**:
  - Models: Generates triangle strips for better performance
  - Textures: Applies platform-specific compression and palette optimization
- **Force full recompile**: Ignores cached intermediate files, rebuilds everything
- **Overwrite on decompile**: Replaces existing extracted files without asking

#### Detailed Workflow

**Decompiling OBJECTS (Characters, Enemies, Items):**

1. Extract the game's WAD.BIN using GDL Archive Tool first
2. In Crucible, click "Select objects dir"
3. Navigate to the folder containing `OBJECTS.PS2`, `OBJECTS.NGC`, or similar
4. Choose your platform target (must match the file)
5. Select output formats (OBJ for models, PNG for textures, YAML for metadata)
6. Click "Decompile objects"

**Results:**
```
your_folder/
├── OBJECTS.PS2                    # Original cache file
├── assets/
│   └── source/                    # Extracted editable files
│       ├── models/
│       │   ├── warrior.obj        # 3D model
│       │   ├── warrior.mtl        # Material file
│       │   ├── dragon.obj
│       │   └── ...
│       ├── bitmaps/
│       │   ├── warrior_skin.png   # Texture files
│       │   ├── dragon_scales.png
│       │   └── ...
│       ├── warrior.yaml           # Metadata (stats, properties)
│       ├── dragon.yaml
│       └── ...
```

**Editing Assets:**

- **Models**: Open `.obj` files in Blender, Maya, 3DS Max, etc.
  - Keep vertex count reasonable (PS2 has limits)
  - Maintain UV mapping for textures
  - Save back as OBJ format

- **Textures**: Edit `.png` files in Photoshop, GIMP, etc.
  - Dimensions must be powers of 2 (64, 128, 256, 512, etc.)
  - Keep original dimensions for best results
  - Alpha channel supported (for transparency)

- **Metadata**: Edit `.yaml` files in any text editor
  - Contains object stats, animations, bounding boxes
  - Be careful with data types (integers vs floats)
  - Incorrect values can crash the game

**Compiling OBJECTS:**

1. After editing assets in `assets/source/`
2. Click "Select objects dir" (same folder as before)
3. Choose platform target
4. Enable optimizations if desired
5. Click "Compile objects"

**Results:**
```
your_folder/
├── OBJECTS.PS2                    # Updated cache file (ready for game)
├── assets/
│   ├── source/                    # Your edited files
│   └── cache/                     # Platform-specific compiled assets
│       ├── models/
│       │   ├── warrior.g3p        # PS2 model cache
│       │   ├── dragon.g3p
│       │   └── ...
│       └── bitmaps/
│           ├── warrior_skin.gtp   # PS2 texture cache
│           └── ...
```

**Working with Messages:**

ROM files contain various game data organized into "lumps" (sections):
- **FONT** - Font bitmaps
- **TEXT/STRS** - Game text and messages
- **PDAT** - Player character data (stats, effects, attacks)
- **ITEM** - Shop items and properties
- **ENMY** - Enemy types and behaviors
- **CAMS** - Camera settings
- **And more...**

1. Click "Select messages dir"
2. Choose folder containing `*.ROM` files (like `BATTLE.ROM`, `CASTLE.ROM`, `ARC.ROM`)
3. Click "Decompile messages"
4. Edit the extracted YAML files
5. Click "Compile messages" to rebuild

---

### GDL Archive Tool - Game Archive Manager

**Purpose:** GDL Archive Tool works with **game archives** (WAD.BIN for PS2, HDD for Arcade) to extract and rebuild the container files that hold all cache files.

**What it does:**
- Extracts entire WAD.BIN archives to folders
- Rebuilds WAD.BIN from folders
- Extracts Arcade HDD images
- Handles compression/decompression automatically
- Preserves or regenerates internal file tables

**When to use it:**
- You want to extract cache files from a game disc image
- You've modified cache files and need to rebuild WAD.BIN for injection
- You're working with Arcade HDD images

**The relationship:**
```
Game Disc
  └── WAD.BIN (archive)              ← GDL Archive Tool works here
       ├── OBJECTS.PS2 (cache)       ← Crucible works here
       ├── WORLDS.PS2 (cache)        ← Crucible works here
       ├── BATTLE.ROM (cache)        ← Crucible works here
       ├── ARC.ROM (cache)           ← Crucible works here
       ├── *.IRX (system files)
       ├── *.VBK (voice/audio)
       └── ... (other game files)
```

#### Running GDL Archive Tool

```bash
python GDL_Archive_Tool.py
```

#### Understanding PS2 WAD.BIN

WAD.BIN is a compressed archive format that stores files with:
- **File headers** - Index of all files with offsets and sizes
- **Path hashing** - Filenames are hashed to 32-bit integers
- **Optional compression** - zlib compression per-file
- **Internal name table** - Optional embedded filename list
- **Sector alignment** - Files aligned to 2048-byte sectors

#### Settings Explained

**WAD.BIN compression level:**
- **0 (No compression)**: Fastest compilation, largest file size
  - Use for testing iterations
- **1 (fastest, largest size)**: Minimal compression, fast
  - Good for quick rebuilds
- **6 (best size for speed)**: Default balanced option
  - Recommended for final builds
- **9 (slowest, smallest size)**: Maximum compression
  - Use when file size is critical

**HDD disc select:**
- Arcade version spans multiple disc images
- **Disc 0**: Main game data
- **Disc 1**: Additional levels/content
- **Disc 2**: More levels/content
- Choose which disc to extract/work with

**Use filenames built into WAD.BIN:**
- When enabled: Reads filename table from WAD.BIN (accurate names)
- When disabled: Uses retail filename list (may be incomplete)
- Recommended: Keep enabled

**Use WAD.BIN file compress list:**
- Some files shouldn't be compressed (already compressed, or hurt performance)
- Reads list from WAD.BIN to preserve compression decisions
- Generally keep enabled

#### Detailed Workflow

**Extracting WAD.BIN (Getting cache files from game):**

1. Obtain WAD.BIN from game disc
   - Extract from ISO using tools like 7-Zip, UltraISO, or WinRAR
   - Usually in the root directory of the disc

2. Launch GDL Archive Tool
3. Click "Decompile HDD/WAD.BIN"
4. Select your WAD.BIN file
5. Choose destination folder (creates one if needed)
6. Tool will extract all files with proper names and extensions

**Results:**
```
extracted_wad/
├── OBJECTS.PS2          # Character/item cache files
├── OBJECTS.NGC
├── WORLDS.PS2           # Level cache files
├── TEXDEF.PS2           # Texture definitions
├── ANIM.PS2             # Animation data
├── BATTLE.ROM           # Battle level data
├── CASTLE.ROM           # Castle level data
├── ARC.ROM              # Player character data (Archer)
├── WAR.ROM              # Player character data (Warrior)
├── *.IRX                # PS2 system modules
├── *.VBK                # Voice/audio banks
└── ...
```

Now you can use **Crucible** on these cache files!

**Rebuilding WAD.BIN (Putting modified files back):**

1. After modifying cache files with Crucible
2. Click "Compile WAD.BIN"
3. Select the folder containing your files
4. Choose where to save new WAD.BIN
5. Set compression level (6 recommended)
6. Tool will rebuild WAD.BIN with all files

**Injecting back into game:**
- Replace WAD.BIN in your game ISO using ISO editing tools
- Tools: UltraISO, PowerISO, or mkisofs
- Test in emulator (PCSX2) or real hardware

**Working with Arcade HDD:**

Arcade version uses raw hard drive images instead of WAD.BIN.

**If you have CHD (compressed HDD):**
```bash
# First convert CHD to raw HDD using chdman (from MAME)
chdman extracthd -i gauntlet.chd -o gauntlet.hdd
```

**Extract from HDD:**
1. Click "Decompile HDD/WAD.BIN"
2. Select the `.hdd` file (raw, not CHD)
3. Choose destination folder
4. Select disc number (0, 1, or 2)
5. Files will be extracted with directory structure preserved

**Note:** Rebuilding Arcade HDD is not yet implemented.

---

### Legend Viewer - 3D Asset Viewer

**Purpose:** Real-time 3D visualization of game assets using the Panda3D engine. Preview worlds, characters, and objects without loading them in the actual game.

**What it does:**
- Loads and displays game worlds in 3D
- Views character models with animations
- Displays items and props
- Shows collision meshes
- Supports texture animations
- Free camera movement

**When to use it:**
- Previewing extracted assets
- Verifying model/texture modifications
- Exploring game levels
- Checking collision meshes
- Studying animation data

#### Running Legend Viewer

```bash
python LegendViewer.py
```

#### Interface Overview

The viewer has two main windows:
1. **Main Window** - 3D viewport with embedded Panda3D renderer
2. **Scene Controls** - (Optional) Control panel for scene navigation
3. **Animation Controls** - (Optional) Animation playback controls

#### Loading Assets

**Loading Worlds:**
1. File → Load world (Ctrl+L)
2. Select folder containing `WORLDS.PS2` or `WORLDS.NGC`
3. World appears in 3D viewport
4. Use camera controls to navigate

**Loading Objects/Actors:**
1. File → Load objects (Ctrl+O)
2. Select folder containing `OBJECTS.PS2` or `OBJECTS.NGC`
3. Initially shows first actor (character)
4. Use Tab to switch between types:
   - **Worlds** - Game levels
   - **Actors** - Playable characters, enemies
   - **Objects** - Items, props, containers

#### Navigation

**Scene Type Switching:**
- `Tab` - Cycle through World/Actor/Object modes
- `Up/Down arrows` - Cycle through resource sets (e.g., different characters)
- `Left/Right arrows` - Cycle through individual items in a set

**Player Count:**
- `` ` `` (backtick) - No players
- `1-4` - Set number of players (spawns character models)

**Camera Controls:**
- `W` - Move forward
- `S` - Move backward
- `A` - Strafe left
- `D` - Strafe right
- `Mouse movement` - Look around
- `Mouse scroll wheel` - Adjust movement speed

**Scene Controls Window (Window → Scene controls):**
- **Type dropdown** - World/Actor/Object
- **Group dropdown** - Select resource set
- **Asset dropdown** - Select specific item
- **Field-of-view slider** - Adjust camera FOV
- **Reset position** - Reset camera to default position
- **Reset rotation** - Reset camera to default angle
- **Player count slider** - 0-4 players

#### Visualization Features

**Debug Toggles:**
- `F1` - Toggle world geometry (show/hide level mesh)
- `F2` - Toggle world collision (show collision mesh)
- `F3` - Toggle item geometry (show/hide item meshes)
- `F4` - Toggle item collision (item collision meshes)
- `F5` - Toggle items visible (all items)
- `F6` - Toggle hidden items visible (show secret items)
- `F7` - Toggle container items (items inside chests, etc.)
- `F8` - Toggle collision grid (visual grid overlay)
- `F9` - Toggle framerate counter
- `F11` - Toggle textures (wireframe with/without textures)
- `F12` - Toggle wireframe mode

**Animation Controls:**
- `Space` - Play/pause animations
- `Backspace` - Reset animation timer to beginning
- Window → Animation controls (advanced panel, mostly TODO)

#### Use Cases

**Verifying Model Edits:**
1. Decompile OBJECTS.PS2 with Crucible
2. Edit model in Blender, export as OBJ
3. Recompile with Crucible
4. Load in Legend Viewer to verify changes

**Exploring Collision Meshes:**
1. Load world in Legend Viewer
2. Press `F2` to show collision mesh
3. Press `F1` to hide visual geometry
4. Navigate to see collision-only view

**Studying Level Layout:**
1. Load world
2. Use free camera to fly through level
3. Toggle items (`F5`) to see placement
4. Toggle hidden items (`F6`) to find secrets

**Previewing Textures:**
1. Load actor or object
2. Press `F11` to toggle textures on/off
3. Press `F12` for wireframe view
4. Verify UV mapping and texture quality

## Understanding the Workflow

### Complete Modding Pipeline

Here's the full workflow from game disc to modified game:

**Phase 1: Extraction**
```
1. Game ISO → Extract WAD.BIN
   Tool: ISO extractor (7-Zip, UltraISO)

2. WAD.BIN → Extract cache files (OBJECTS.PS2, *.ROM, etc.)
   Tool: GDL Archive Tool (Decompile)

3. Cache files → Extract assets (OBJ, PNG, YAML)
   Tool: Crucible (Decompile)
```

**Phase 2: Modification**
```
4. Edit OBJ models in Blender/Maya
5. Edit PNG textures in Photoshop/GIMP
6. Edit YAML metadata in text editor
7. Preview in Legend Viewer (optional)
```

**Phase 3: Compilation**
```
8. Assets → Compile to cache files
   Tool: Crucible (Compile)

9. Cache files → Rebuild WAD.BIN
   Tool: GDL Archive Tool (Compile)

10. WAD.BIN → Inject into ISO
    Tool: ISO editor (UltraISO, mkisofs)
```

**Phase 4: Testing**
```
11. Test in emulator (PCSX2) or real hardware
12. Iterate as needed
```

### Quick Reference: Which Tool When?

| Task | Tool | Direction |
|------|------|-----------|
| Extract files from WAD.BIN | GDL Archive Tool | Decompile |
| Extract models from OBJECTS.PS2 | Crucible | Decompile |
| Edit 3D models | External (Blender, etc.) | - |
| Edit textures | External (Photoshop, etc.) | - |
| Preview changes | Legend Viewer | View |
| Rebuild OBJECTS.PS2 | Crucible | Compile |
| Rebuild WAD.BIN | GDL Archive Tool | Compile |
| Inject WAD.BIN into ISO | External (UltraISO, etc.) | - |

## File Structure

### Standard Directory Layout

After full extraction and decompilation:

```
gauntlet_mod/
├── original_wad/
│   └── WAD.BIN                        # Original game archive
│
├── extracted_wad/                     # GDL Archive Tool output
│   ├── OBJECTS.PS2                    # Cache files
│   ├── WORLDS.PS2
│   ├── TEXDEF.PS2
│   ├── ANIM.PS2
│   ├── *.ROM
│   └── ... (other files)
│
└── extracted_wad/
    └── assets/                        # Crucible output
        ├── source/                    # Editable assets (EXPORT)
        │   ├── models/
        │   │   ├── warrior.obj
        │   │   ├── warrior.mtl
        │   │   └── ...
        │   ├── bitmaps/
        │   │   ├── warrior_skin.png
        │   │   └── ...
        │   ├── warrior.yaml           # Metadata
        │   └── ...
        │
        └── cache/                     # Compiled assets (IMPORT)
            ├── models/
            │   ├── warrior.g3p        # Platform cache
            │   └── ...
            └── bitmaps/
                ├── warrior_skin.gtp
                └── ...
```

### Cache File Extensions

**Model Cache:**
- `.g3p` - PlayStation 2 (PS2)
- `.g3n` - GameCube (NGC)
- `.g3x` - Xbox
- `.g3a` - Arcade

**Texture Cache:**
- `.gtp` - PlayStation 2
- `.gtn` - GameCube
- `.gtx` - Xbox
- `.gta` - Arcade

**Other Cache:**
- `.g4d` - Animation cache
- `.g3c` - Collision cache
- `.ncc` - NCC table cache (GameCube)

**Main Cache Files:**
- `.ps2` - PS2 main cache (OBJECTS, WORLDS, TEXDEF, ANIM)
- `.ngc` - GameCube main cache
- `.rom` - Generic cache / Arcade main cache

## Supported Formats

### Import Formats (Source Assets)

**3D Models:**
- Wavefront OBJ (.obj + .mtl)
- Must include vertex positions, normals, and UV coordinates
- Collada DAE support planned but not yet implemented

**Textures:**
- PNG (.png) - Recommended for editing
- Targa TGA (.tga) - Good alpha channel support
- DirectDraw Surface DDS (.dds) - Pre-compressed formats

**Metadata:**
- YAML (.yaml, .yml) - Human-readable
- JSON (.json) - Machine-readable

### Export Formats (Game Cache)

**PlayStation 2:**
- Models: G3P format with VIF/DMA code generation
- Textures: GTP format with PS2 texture buffer packing
- Special: TEXDEF.PS2 texture definition cache
- Supports palettized textures, swizzling, mipmaps

**GameCube:**
- Models: G3N format optimized for GameCube
- Textures: GTN format with GameCube-specific compression
- Special: NCC color lookup tables
- Supports ABGR_3555 format (GameCube exclusive)

**Xbox:**
- Models: G3X format
- Textures: GTX format (DirectX-compatible)
- Supports standard DirectX texture formats

**Arcade:**
- Models: G3A format
- Textures: GTA format
- Special arcade texture formats (BGR_565, P_8, etc.)
- Note: Full cache compilation not yet supported

### Archive Formats

**PS2 WAD.BIN:**
- Custom archive format with zlib compression
- Files identified by 32-bit path hash
- Sector-aligned (2048 bytes)
- Supports embedded filename table
- Max ~65000 files

**Arcade HDD:**
- Raw hard disk image
- Multi-disc support (3 discs)
- FAT-like filesystem structure
- CHD (Compressed Hunks of Data) variant supported via chdman

## Platform Support

### PlayStation 2
✅ **Full Support**
- Decompilation: ✓
- Compilation: ✓
- Archive extraction: ✓
- Archive creation: ✓
- Features:
  - VIF/DMA code generation for models
  - PS2 texture buffer packing algorithm
  - TEXDEF cache generation
  - Palette optimization
  - Swizzling support

### GameCube
✅ **Full Support**
- Decompilation: ✓
- Compilation: ✓
- Archive extraction: N/A (different format)
- Archive creation: N/A
- Features:
  - GameCube-specific texture formats
  - NCC color table generation
  - ABGR_3555 format support
  - Optimized triangle strips

### Xbox
✅ **Full Support**
- Decompilation: ✓
- Compilation: ✓
- Archive extraction: N/A (different format)
- Archive creation: N/A
- Features:
  - DirectX-compatible formats
  - Standard texture compression
  - Optimized for Xbox hardware

### Arcade
⚠️ **Partial Support**
- Decompilation: ✓
- Model/texture compilation: ✓
- Cache file compilation: ✗ (not yet supported)
- HDD extraction: ✓
- HDD creation: ✗ (not yet implemented)
- Features:
  - Multi-disc HDD support
  - CHD conversion support (via chdman)
  - Arcade-specific texture formats
  - Directory tree parsing

## Known Limitations

**Compilation:**
- ❌ Arcade cache file compilation not supported (models/textures work individually)
- ❌ World collision compilation incomplete
- ⚠️ Worlds compilation disabled in Crucible GUI

**File Formats:**
- ❌ Only OBJ format supported for models (no DAE/FBX)
- ⚠️ Some texture formats require specific dimensions

**Archives:**
- ❌ Arcade HDD compilation not implemented
- ❌ CHD files must be converted to raw HDD first
- ⚠️ WAD.BIN filename hashing can cause name collisions

**Platform:**
- ⚠️ GameCube/Xbox archive formats differ from PS2 (no WAD.BIN equivalent)

## Troubleshooting

### "Could not load texdefs" warning
**Cause:** TEXDEF.PS2 file not present in directory
**Solution:** This is normal. Tools will generate best-guess names for textures.
**Impact:** Texture names may be generic (TEX_001, TEX_002, etc.)

### Compilation is very slow
**Solutions:**
- Enable "Use parallel processing" in Crucible settings
- Reduce texture resolution if possible
- Disable "Force full recompile" after first build
- Use faster compression level in GDL Archive Tool (level 1 vs 9)

### Textures look corrupted/wrong colors
**Checks:**
1. Texture dimensions are powers of 2? (64, 128, 256, 512, 1024)
2. Platform target matches the cache file?
3. Texture format appropriate for platform?
4. Alpha channel preserved if needed?

**Solution:** Resize to power-of-2, verify platform setting

### Models don't appear in game
**Checks:**
1. Vertex normals included in OBJ export?
2. UV coordinates present?
3. Model within reasonable polygon count? (PS2 has limits)
4. Compiled for correct platform?

**Solution:** Re-export from 3D editor with normals and UVs

### Legend Viewer crashes on startup
**Cause:** Panda3D not installed or misconfigured
**Solution:**
```bash
pip uninstall panda3d
pip install panda3d
```

### WAD.BIN too large for disc
**Solutions:**
- Use higher compression (level 9)
- Reduce texture resolutions
- Optimize models (lower polygon count)
- Remove unused assets

### Changes don't appear in game
**Checklist:**
1. ✓ Edited source assets in `assets/source/`?
2. ✓ Compiled with Crucible?
3. ✓ Rebuilt WAD.BIN with GDL Archive Tool?
4. ✓ Injected new WAD.BIN into ISO?
5. ✓ Using correct version of game?
6. ✓ Cache cleared in emulator?

## Advanced Topics

### Understanding Texture Formats

The game uses various pixel formats depending on platform:

**Common Formats:**
- `ABGR_8888` - 32-bit true color with alpha
- `XBGR_8888` - 32-bit true color without alpha
- `ABGR_1555` - 16-bit color with 1-bit alpha
- `ABGR_8888_IDX_8` - 8-bit indexed (256 color palette)

**GameCube Exclusive:**
- `ABGR_3555` - 16-bit with 3-bit alpha

**Arcade Specific:**
- `BGR_565` - 16-bit color, no alpha
- `P_8` - 8-bit palette indexed
- `ABGR_4444` - 16-bit, 4-bit alpha

See [gdl/compilation/g3d/constants.py](gdl/compilation/g3d/constants.py#L69-L107) for complete list.

### Parallel Processing

Both Crucible and GDL Archive Tool support multi-core processing:

**How it works:**
- Jobs split across CPU cores
- Files sorted by size for load balancing
- Larger files processed first
- Temp files used for concurrent WAD building

**Performance:** 2-4x faster on modern CPUs (4+ cores)

### Triangle Strip Optimization

When "Optimize models" is enabled:
- Converts triangle soup to triangle strips
- Reduces vertex transformations
- Better cache coherency on PS2/GameCube
- Can reduce file size by 20-40%

**Trade-off:** Slightly longer compile time

### Metadata Structure

YAML files contain object properties:
```yaml
name: "WARRIOR"
model: "warrior.obj"
texture: "warrior_skin.png"
bounding_radius: 32.5
vertex_count: 1024
triangle_count: 1845
animations:
  - idle
  - walk
  - attack
stats:
  health: 100
  speed: 5.0
  attack: 15
```

## License

This project is licensed under the GNU Lesser General Public License v2.1 - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

**Built using:**
- [supyr_struct](https://github.com/Sigmmma/supyr_struct) - Binary structure definitions
- [binilla](https://github.com/Sigmmma/binilla) - Tag editor framework
- [Panda3D](https://www.panda3d.org/) - 3D rendering engine
- [arbytmap](https://github.com/Sigmmma/arbytmap) - Bitmap processing

**Special thanks to the game modding community for reverse engineering Gauntlet: Dark Legacy's formats.**
