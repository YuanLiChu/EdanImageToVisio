# EdanImageToVisio

This folder is an Agent Skill for recreating reference diagrams as editable Microsoft Visio `.vsdx` files. It can be used by Codex-style skill loaders and Claude Code skill loaders that support `SKILL.md`.

It supports a practical workflow:

1. Inspect a reference image, screenshot, Mermaid export, flowchart, sequence diagram, or architecture diagram.
2. Create a JSON drawing plan made of editable primitives.
3. Run the bundled PowerShell script to draw the plan in Visio through COM automation.
4. Save `.vsdx` and optionally export `.emf` preview.

## Requirements

- Windows
- Microsoft Visio
- PowerShell

## Install

Copy the `edan-image-to-visio` folder into your Codex skills directory:

```powershell
$skills = "$env:USERPROFILE\.codex\skills"
New-Item -ItemType Directory -Force -Path $skills | Out-Null
Copy-Item -Recurse -Force ".\edan-image-to-visio" $skills
```

Restart Codex or reload skills.

For Claude Code user-level installation, copy the same folder to:

```powershell
$skills = "$env:USERPROFILE\.claude\skills"
New-Item -ItemType Directory -Force -Path $skills | Out-Null
Copy-Item -Recurse -Force ".\edan-image-to-visio" $skills
```

## Example Prompt

```text
Use $edan-image-to-visio to recreate this reference diagram as an editable Visio file, export a preview, and explain any approximations.
```

## Notes

This skill is not an automatic image vectorizer. It relies on Codex to create a JSON diagram plan, then uses Visio COM automation to generate editable Visio shapes.
