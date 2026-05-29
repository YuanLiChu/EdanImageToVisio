# Visio JSON Plan Schema and QA

Read this file before creating a JSON plan for `scripts/create_visio_from_plan.ps1`.

The plan uses image-like coordinates:

- Origin is the top-left corner.
- `x` grows to the right.
- `y` grows downward.
- `scalePxPerInch` maps pixels to Visio page inches.

## Minimal Example

```json
{
  "page": {
    "name": "Architecture Replica",
    "widthPx": 1440,
    "heightPx": 928,
    "scalePxPerInch": 100
  },
  "referenceImage": "C:/workspace/reference.png",
  "shapes": [
    {
      "type": "rect",
      "x1": 20,
      "y1": 90,
      "x2": 1420,
      "y2": 910,
      "text": "",
      "style": {
        "fill": "RGB(255,255,255)",
        "line": "RGB(88,103,150)",
        "weight": 1.4,
        "dash": true
      }
    },
    {
      "type": "text",
      "x1": 590,
      "y1": 35,
      "x2": 850,
      "y2": 75,
      "text": "System Architecture",
      "style": {
        "fontSize": 22,
        "textColor": "RGB(0,153,188)",
        "bold": true
      }
    },
    {
      "type": "line",
      "x1": 300,
      "y1": 200,
      "x2": 430,
      "y2": 200,
      "arrow": "end",
      "style": {
        "weight": 1.2
      }
    },
    {
      "type": "polyline",
      "points": [[300, 250], [300, 310], [430, 310]],
      "arrow": "end",
      "style": {
        "weight": 1.2
      }
    },
    {
      "type": "oval",
      "x1": 500,
      "y1": 220,
      "x2": 550,
      "y2": 270,
      "text": "API",
      "style": {
        "fill": "RGB(231,252,255)",
        "line": "RGB(35,188,211)",
        "fontSize": 8,
        "textColor": "RGB(34,139,155)"
      }
    }
  ]
}
```

## Shape Types

### `rect`

Rectangle. Required fields:

- `x1`, `y1`, `x2`, `y2`

Optional:

- `text`
- `style`

### `text`

Text box drawn as a rectangle with no fill and no border.

Required:

- `x1`, `y1`, `x2`, `y2`
- `text`

Optional:

- `style`

### `oval`

Oval inside a bounding box.

Required:

- `x1`, `y1`, `x2`, `y2`

Optional:

- `text`
- `style`

### `line`

Straight line.

Required:

- `x1`, `y1`, `x2`, `y2`

Optional:

- `arrow`: `none`, `begin`, `end`, or `both`
- `style`

### `polyline`

Multi-segment line.

Required:

- `points`: array of `[x, y]`

Optional:

- `arrow`: applied to the last segment
- `style`

## Style Fields

- `fill`: Visio formula such as `RGB(255,255,255)`.
- `line`: Visio formula such as `RGB(35,35,35)`.
- `weight`: line weight in points.
- `dash`: boolean.
- `noFill`: boolean.
- `noLine`: boolean.
- `fontSize`: text size in points.
- `textColor`: Visio formula such as `RGB(40,40,40)`.
- `bold`: boolean.
- `align`: Visio horizontal alignment integer. Use `0` for left and `1` for centered.

## QA Checklist

- Page aspect ratio matches the reference.
- Large containers and boundaries are placed first.
- Text is readable and not wildly overflowing.
- Arrows point in the correct direction.
- Dashed boundaries are visually clear.
- Preview export can be opened without Visio.
- `.vsdx` contains editable shapes rather than only one bitmap.
- If a reference image is included, it is on a separate page from the editable drawing.
- All output and image paths passed to Visio are absolute paths.
- Reference image paths use ASCII-only names when possible.
