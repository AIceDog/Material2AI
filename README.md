# Material2AI Documentation

Material2AI is an Unreal Engine editor plugin for exporting Material and
Material Function graphs into structured text reports that can be used for AI
assistance, documentation, debugging, and technical review.

The plugin is designed for editor workflows. It does not modify source assets
and does not add runtime gameplay functionality.

## Supported Unreal Engine Version

Tested with:

- Unreal Engine 5.7.4
- Windows / Win64 editor workflow

Current plugin descriptor:

- Module type: Editor
- Supported platform: Win64
- Runtime replication: No
- Blueprint assets included: No

## What Material2AI Exports

Material2AI can export:

- Material root output trees
- Material Function output trees
- Connected node chains
- Default values for active but unconnected material outputs
- Default values for unconnected expression inputs when available
- Referenced Material Functions
- Optional Material Details
- Optional Material Function Details
- Optional Material Node Properties
- Optional Material Function Node Properties

The output is intentionally text-first so it can be pasted into AI tools,
technical notes, bug reports, or internal documentation.

## Installation

Install the plugin into a project-level Plugins folder:

```text
<YourProject>/Plugins/Material2AI/
```

The folder should contain:

```text
Material2AI/
  Material2AI.uplugin
  Resources/
  Source/
  README.md
```

Then:

1. Open the Unreal Engine project.
2. Let Unreal compile the plugin if prompted.
3. Enable `Material2AI` from the Plugins window if it is not already enabled.
4. Restart the editor if Unreal asks for a restart.

For source builds, make sure Visual Studio and the required Unreal C++ build
tools are installed.

## Basic Usage

Material2AI has two main editor entry points.

### Content Browser Export

1. Select one or more supported assets in the Content Browser.
2. Right-click the selection.
3. Choose `Material2AI: Export`.
4. The Material2AI export window opens with the generated report.

Supported asset types include:

- Material
- Material Function
- Material Function Instance
- Material Function Material Layer
- Material Function Material Layer Blend

### Material Editor Export

1. Open a Material or Material Function in the Unreal Editor.
2. Click the `Material2AI` toolbar button.
3. The export window opens for the asset currently being edited.

The toolbar path is useful when you are already inspecting a graph and want to
generate a report without returning to the Content Browser.

## Export Window

The export window provides:

- `Refresh`: Regenerates the report from the current asset references.
- `Copy`: Copies the visible report text to the clipboard.
- `Save .md`: Saves the visible report under the project `Saved/Material2AI`
  folder.
- `Material > Details`: Includes reflected Material details.
- `Material > Node Properties`: Includes reflected Material expression
  properties.
- `Material Functions > Details`: Includes reflected Material Function details.
- `Material Functions > Node Properties`: Includes reflected function
  expression properties.
- `Material Functions > Expand`: Expands referenced Material Functions into
  their own report sections.
- `Depth`: Controls how deeply nested Material Functions are expanded.

Most detailed property sections are optional because large materials can produce
very long reports. The default output focuses on graph structure first.

## Output Structure

A typical export is organized by asset.

For a Material, the report can include:

```text
Material
  Graph Tree
  Details
  Node Properties
  Used Material Functions
```

For each referenced Material Function, the report can include:

```text
Material Function
  Interface
  Graph Tree
  Details
  Node Properties
  Used Material Functions
```

This keeps the top-level Material and each Material Function self-contained,
which makes the report easier to read and easier to send to an AI assistant.

## Example Output

Example tree-style output:

```text
* Material M_Example
    * Root Outputs
        |-- Base Color:
        |   L-- N0.RGB Constant3Vector [0,0,0]
        |
        |-- Roughness:
        |   L-- N1.R Multiply
        |       |-- A:
        |       |   L-- N2.R ScalarParameter [Roughness]
        |       L-- B:
        |           L-- Default Value: 0.5
        |
        L-- Normal:
            L-- Default Value: 0.0,0.0,1.0
```

Connector meaning:

- `|--` means this branch has more sibling branches after it.
- `L--` means this is the last branch at that level.
- A blank connector line between root outputs is used only for readability.

## Material Function Expansion

When Material Function expansion is enabled, Material2AI detects function call
nodes and adds the referenced Material Functions to the report.

Expansion depth is limited to keep exports responsive and avoid infinite loops
from recursive or shared function references.

Depth behavior:

- `0`: Do not expand referenced functions.
- `1`: Expand functions directly referenced by the selected asset.
- `2`: Expand direct functions and one nested level.
- Maximum allowed depth: 8

Already visited functions are tracked during export so repeated references do
not produce infinite recursion.

## Saved Reports

When `Save .md` is clicked, Material2AI saves the current visible text to:

```text
<YourProject>/Saved/Material2AI/
```

The visible text is saved rather than only the generated text. This means users
can manually trim or annotate the report in the export window before saving.

## Stability And Safety Notes

Material2AI is designed to avoid editor crashes during export.

Safety behavior includes:

- Unsupported assets are ignored or reported instead of crashing.
- Null and invalid asset references are skipped.
- Preview/transient editor materials are filtered where possible.
- Material Function recursion depth is clamped.
- Large reports are truncated before they become too heavy for the editor.
- Very large graph/property sections include warning lines in the report.
- Save file names are sanitized before writing to disk.

The plugin reads graph data and reflected properties. It does not edit, compile,
resave, or otherwise modify Material or Material Function assets.

## Packaging Notes

When packaging the plugin with Unreal's `BuildPlugin` workflow, use an
ASCII-only output path.

Recommended package paths:

```text
C:/UEPluginPackages/Material2AI
F:/UEPluginPackages/Material2AI
```

Avoid package output paths containing non-ASCII characters, such as Chinese
characters, because Unreal Automation Tool / build tooling may produce garbled
intermediate paths on some Windows setups.

For distribution, do not include local generated folders such as:

```text
Binaries/
Intermediate/
Saved/
DerivedDataCache/
```

A clean source distribution should normally include:

```text
Material2AI.uplugin
README.md
Resources/
Source/
```

## Technical Details

Plugin module:

```text
Material2AI - Editor
```

Primary implementation areas:

- `FMaterial2AIModule`: Registers editor menu and toolbar entry points, resolves
  selected editor assets, and opens the export window.
- `SMaterial2AIOutputWindow`: Slate UI for previewing, refreshing, copying, and
  saving generated reports.
- `FMaterial2AIExporter`: Walks Material and Material Function graph data and
  generates the structured text report.
- `FMaterial2AIExportSettings`: Stores user-controlled export options.

Important Unreal Engine systems used:

- ToolMenus
- Content Browser asset context menus
- Material Editor toolbar extension
- Slate UI
- UObject reflection
- Material expression graph APIs

## Limitations

- Material2AI is editor-only and is not intended for packaged game runtime use.
- Current release targets Win64 editor workflows.
- Very large graphs may be summarized or truncated to keep the editor
  responsive.
- Reflected Details and Node Properties can be verbose; leave those options off
  when preparing compact AI prompts.
- The exported report is a structural description, not a replacement for Unreal
  Engine's material compiler or shader code output.

## Troubleshooting

### The plugin does not appear in the editor

Check that the folder is installed at:

```text
<YourProject>/Plugins/Material2AI/
```

Then regenerate project files, rebuild the project, and enable the plugin in
the Plugins window.

### Packaging fails with a missing SharedPCH source file

Use an ASCII-only package output path.

For example:

```text
C:/UEPluginPackages/Material2AI
```

This can happen if the package destination contains non-ASCII characters and
the build toolchain garbles the temporary HostProject path.

### The report is too long for an AI prompt

Try disabling:

- Material Details
- Material Function Details
- Material Node Properties
- Material Function Node Properties

You can also reduce the Material Function expansion depth.

### Some inputs show default values instead of nodes

This means the input was not connected to another expression, but the exporter
found a usable editor default value for that input.

### A Material Function appears only as a reference

Enable `Material Functions > Expand`, then increase `Depth` if the function is
nested deeper than the current limit.

## FAQ

### Does Material2AI modify my materials?

No. Material2AI only reads editor graph data and generates a text report.

### Can I use it in a packaged game?

No. Material2AI is an editor plugin. It is meant for Unreal Editor workflows.

### Does it require an AI service or API key?

No. Material2AI only generates structured text. You can paste or save that text
and use it with the AI tool of your choice.

### Does it support Material Functions?

Yes. It can export Material Function assets directly and can expand referenced
Material Functions from a Material graph.

### Where are saved reports stored?

Saved reports are written under:

```text
<YourProject>/Saved/Material2AI/
```

## Support

For support, include:

- Unreal Engine version
- Material2AI version
- Whether the issue happens from the Content Browser or Material Editor toolbar
- The selected asset type
- Any warnings shown in the generated report
- Relevant Unreal Output Log messages

