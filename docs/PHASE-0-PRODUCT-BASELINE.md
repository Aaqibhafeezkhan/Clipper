# Clipper — Phase 0 Product Baseline

## Purpose

This document records the baseline for Clipper before product behavior is changed. It is the reference point for subsequent phases covering correctness, accessibility, compatibility, performance, polish, testing, and release readiness.

## Product contract

Clipper is a client-side browser utility for adding a white border around an image or removing a border by cropping the image. The current implementation performs the work locally in the browser and produces a PNG download.

### Primary user flow

1. Choose an image through the file picker or drag and drop it onto the drop area.
2. Adjust the Border / crop percentage.
3. Choose Add white border or Remove border.
4. Inspect the processed preview and resulting dimensions.
5. Download the processed image as a PNG.

## Current implementation baseline

The application is currently a single `index.html` containing the markup, CSS, and JavaScript.

The browser APIs currently used are:

- File input and drag-and-drop events for image selection.
- `FileReader` to load the selected file as a data URL.
- `Image` to decode the source image and obtain its intrinsic dimensions.
- `HTMLCanvasElement` and `CanvasRenderingContext2D` for border addition and cropping.
- `canvas.toBlob()` for PNG output.
- `URL.createObjectURL()` for the processed preview and download link.

There is no server-side image-processing dependency in the current implementation.

## Current supported behavior

### Input

The UI accepts image files through the browser file picker and drag-and-drop flow. The file input advertises image formats through `accept="image/*"`; the visible copy currently names JPG, PNG, and WebP.

### Border addition

For a source image with dimensions `W × H` and a selected percentage `p`, the current implementation calculates:

- Horizontal border per side: `round(W × p)`
- Vertical border per side: `round(H × p)`
- Output width: `W + 2 × horizontal border`
- Output height: `H + 2 × vertical border`

The new canvas is filled white and the source image is drawn at the calculated inset.

### Border removal

For the same source dimensions and percentage, the implementation crops:

- `round(W × p)` pixels from the left and right.
- `round(H × p)` pixels from the top and bottom.

The crop dimensions are clamped to a minimum of one pixel before drawing to the output canvas.

### Output

Processed output is encoded as PNG. The download filename is based on the original filename with `-clipped.png` appended.

## Current UX baseline

- Desktop-first centered layout with a responsive single-column fallback below 600px.
- Drag-and-drop target with hover/drag visual feedback.
- Percentage range from 1% through 50%, defaulting to 20%.
- Buttons are disabled until an image has been loaded.
- The preview uses a checkerboard-style background to make transparent areas visually apparent.
- Source dimensions are shown after image loading and updated to processed dimensions after transformation.
- No explicit processing spinner, error message, or success/status announcement currently exists.
- No explicit reset/remove-image control currently exists.

## Known constraints and risks

1. **Input validation is minimal.** The current loader checks for an image MIME type but does not communicate invalid selections to the user.
2. **Image memory can grow.** Source data URLs and generated object URLs can remain referenced longer than necessary during repeated operations.
3. **Large images can block the main thread.** Canvas decoding and rendering occur synchronously in the browser UI flow.
4. **Transparency behavior is not an explicit product contract.** Added borders are intentionally white, while the output format is always PNG.
5. **Format support is browser-dependent.** Actual decoding support comes from the browser's image implementation rather than an application-level format matrix.
6. **Repeated processing changes the current source context.** After processing, the preview is replaced with the generated object URL while the original decoded source remains the processing source.
7. **The current percentage semantics are asymmetric with the common meaning of total-image percentage.** The selected percentage is applied independently to each side, so removal can approach the full source dimension at the upper range.
8. **Accessibility coverage is incomplete.** The visible drop area and controls need a deliberate keyboard, focus, labeling, and status strategy.
9. **There is no automated test harness.** Core transformation logic currently lives directly in the page script.
10. **The project has no formal browser compatibility policy.** Supported browsers and required APIs need to be defined before release hardening.

## Product non-goals

For the current roadmap, Clipper should remain a focused image utility rather than becoming a general-purpose image editor. The following are non-goals unless a future phase explicitly changes scope:

- Full photo editing.
- Filters, effects, drawing, or retouching.
- Cloud uploads or server-side storage.
- User accounts or authentication.
- Persistent user image libraries.
- Server-side analytics that require uploading image content.
- Automated deployment workflows.

## Privacy principles

- Image processing should remain local to the browser whenever practical.
- Image content should not be uploaded to a backend merely to perform border or crop operations.
- Object URLs and in-memory image data should have bounded lifetimes where practical.
- Any future telemetry must be evaluated separately and must not require image content.

## Engineering principles for later phases

### Correctness

Transformation dimensions, crop coordinates, image orientation, transparency, and output format must be explicit and testable.

### Accessibility

Every interactive flow should work without a mouse, expose meaningful labels and status information, and provide visible focus states.

### Performance

Avoid unnecessary image copies, release temporary browser resources, and keep the UI responsive for realistic image sizes.

### Compatibility

Document the minimum browser/API baseline and provide graceful behavior when a required capability is unavailable.

### Maintainability

Separate transformation logic from presentation concerns as complexity grows, without introducing a framework or abstraction solely for its own sake.

### User trust

Processing behavior, output dimensions, privacy expectations, and failures should be understandable to a user without requiring developer knowledge.

## Release-readiness criteria

Clipper should not be considered release-ready until the project can demonstrate:

- Predictable border and crop calculations across representative image dimensions.
- Clear handling of invalid and unsupported input.
- Keyboard-accessible controls and meaningful focus states.
- Useful feedback for loading, processing, success, and failure states.
- Defined browser compatibility and image-format expectations.
- Safe handling of object URLs and large in-memory image data.
- A practical automated test strategy for transformation behavior.
- A documented manual deployment path.
- No unnecessary external service dependency for core image processing.

## Phase roadmap

| Phase | Focus | Outcome |
| --- | --- | --- |
| 0 | Product baseline and engineering foundation | This baseline and release criteria |
| 1 | Core image-processing correctness | Predictable transformations and edge-case handling |
| 2 | UX and accessibility | Keyboard-friendly, accessible, resilient interaction |
| 3 | Image formats and browser compatibility | Explicit support matrix and graceful degradation |
| 4 | Performance and memory safety | Better behavior for repeated and large-image processing |
| 5 | Product polish | Refined preview, controls, and feedback |
| 6 | Testing and release readiness | Repeatable validation and documented release flow |
| 7 | Final QA | Release-ready product |

## Phase 0 completion criteria

- [x] Current product flow documented.
- [x] Current architecture and browser APIs documented.
- [x] Input/output and transformation semantics documented.
- [x] Known limitations and risks documented.
- [x] Product non-goals and privacy principles documented.
- [x] Engineering and release-readiness criteria defined.
