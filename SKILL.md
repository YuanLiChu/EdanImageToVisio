---
name: edan-image-to-visio
description: Create editable Microsoft Visio diagrams from reference images, screenshots, Mermaid exports, architecture diagrams, flowcharts, sequence diagrams, or hand-written JSON diagram plans. Use when Codex needs to recreate a diagram as editable .vsdx shapes, generate a JSON drawing plan, run Visio COM automation, embed a reference image page, export an EMF preview, or prepare a screen-recording-friendly Visio recreation workflow on Windows.
---

# EdanImageToVisio

Use this skill when the user wants a reference diagram recreated as an editable Microsoft Visio file.

This is not a pixel-perfect image vectorizer. The reliable workflow is:

1. Understand the input diagram or Mermaid/exported image.
2. Create a JSON drawing plan with editable shapes.
3. Run `scripts/create_visio_from_plan.ps1` to generate `.vsdx`.
4. Export an `.emf` preview when useful.
5. Optionally embed the original image on a second Visio page for comparison.

## Requirements

- Windows.
- Microsoft Visio installed.
- PowerShell.
- Visio COM automation available.

Run this first when working on a new machine:

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File scripts/check_visio_environment.ps1 -TryCom
```

If `comStartup` is not `success` or `VISIO.EXE` is missing, explain that `.vsdx` generation cannot be verified on this machine.

## Input Modes

Choose the simplest mode that fits the request.

- **JSON plan to Visio**: If the user already has a plan, run the script directly.
- **Image or screenshot to Visio**: Inspect the image, create a JSON plan manually, then generate `.vsdx`.
- **Mermaid to Visio**: Prefer using the Mermaid source if provided. If only a Mermaid PNG is available, recreate the visible structure as a JSON plan.
- **Reference-only Visio**: If the user only needs the image inside Visio, create a plan with `referenceImage`; be clear that this is not editable diagram extraction.

## JSON Plan Workflow

Read `references/plan_schema_and_qa.md` before writing a new plan.

Use absolute paths for `PlanPath`, `OutVsdx`, `OutEmf`, and `referenceImage`. Visio COM can fail on relative output paths, and image import may fail on paths with non-ASCII characters. If needed, copy the reference image to an ASCII-only workspace path first.

Example command:

```powershell
$root = (Get-Location).Path
$plan = Join-Path $root "output/diagram-plan.json"
$vsdx = Join-Path $root "output/diagram.vsdx"
$emf = Join-Path $root "output/diagram.emf"
powershell -NoProfile -ExecutionPolicy Bypass -File scripts/create_visio_from_plan.ps1 -PlanPath $plan -OutVsdx $vsdx -OutEmf $emf
```

## Drawing Guidance

- Use editable Visio primitives instead of pasting one bitmap.
- Build large containers first, then inner sections, then text, then connectors.
- Preserve the diagram's hierarchy and flow more than every pixel.
- Use separate `text` shapes for dense labels when text needs better placement.
- Use dashed rectangles for module boundaries.
- Use `line` for simple arrows and `polyline` for elbow arrows or loop-like arrows.
- Use `referenceImage` to add a second page containing the original diagram.
- For complex screenshots, create a first pass that captures structure, then iterate visually after exporting a preview.

## Validation

After generation, verify at least:

- The `.vsdx` file exists and is non-empty.
- The `.emf` preview exists when requested.
- The package contains `visio/pages/page1.xml`.
- Page 1 contains multiple `Shape` nodes, proving it is not only a pasted bitmap.

PowerShell package check:

```powershell
$vsdx = "C:/path/to/diagram.vsdx"
Add-Type -AssemblyName System.IO.Compression.FileSystem
$zip = [System.IO.Compression.ZipFile]::OpenRead($vsdx)
try {
  $page1 = $zip.GetEntry("visio/pages/page1.xml")
  $reader = New-Object System.IO.StreamReader($page1.Open())
  try { [xml]$xml = $reader.ReadToEnd() } finally { $reader.Close() }
  $nsm = New-Object System.Xml.XmlNamespaceManager($xml.NameTable)
  $nsm.AddNamespace("v","http://schemas.microsoft.com/office/visio/2012/main")
  [pscustomobject]@{
    size = (Get-Item $vsdx).Length
    shapeCount = $xml.SelectNodes("//v:Shape", $nsm).Count
    textNodeCount = $xml.SelectNodes("//v:Text", $nsm).Count
  }
} finally {
  $zip.Dispose()
}
```

## Known Limits

- The script supports `rect`, `text`, `oval`, `line`, and `polyline`.
- It does not parse Mermaid syntax by itself.
- It does not perform OCR or automatic image vectorization by itself.
- It does not create native Visio dynamic connectors; lines are editable line shapes.
- Rounded rectangles, gradients, icons, and exact font rendering may need manual refinement.
- PNG import can fail for paths with non-ASCII characters; copy images to an ASCII path first.

## Deliverables

For a normal task, provide:

- The `.vsdx` path.
- The JSON plan path.
- The preview path when exported.
- A short note about what is editable and what was approximated.
