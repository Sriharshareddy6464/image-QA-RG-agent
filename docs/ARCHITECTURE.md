# IQARG — Architecture

> Status: Provisional MVP architecture. Changes require approval from the architecture owner.

## 1. Architectural objective

Separate the user interface, application API, vision/segmentation, image processing, and image generation so model implementations can be replaced without rewriting the product workflow.

MVP v1 runs locally through Docker Compose. AMD Cloud GPU deployment is deferred.

## 2. Logical architecture

```text
React + Vite frontend
  - Upload, canvas, click selection
  - Mask preview/refinement, prompt
  - Progress, before/after, accept/retry
          | HTTP API + WebSocket
          v
FastAPI backend
  - Validation and job orchestration
  - Coordinate contract and storage references
  - Segmentation and generation adapters
       |                         |
       v                         v
Vision / segmentation      Image processing
- Mirage experiment        - OpenCV / Pillow
- Point-mask fallback     - Mask refinement
  (e.g. SAM 2)             - Feather/composite
       |                         |
       +------------+------------+
                    v
             ComfyUI service
             - Inpainting workflow
                    |
                    v
            Local persistent storage
```

This is a logical view, not a requirement to create a separate container for every box. Keep deployment small. Split services only when isolation, dependencies, or GPU memory behavior justify it.

## 3. Components and responsibilities

### 3.1 Frontend

Technology: React, Vite, TypeScript. Canvas library is provisional.

Responsibilities:

- Upload and display an image.
- Maintain displayed image geometry and original-image dimensions.
- Convert pointer coordinates from display space into original-image coordinates.
- Send selection points and user actions to the backend.
- Display candidate and approved masks distinctly.
- Support the agreed mask correction interaction.
- Submit the edit prompt.
- Display job progress.
- Compare original and generated images.
- Allow accept, reject, and retry actions.

The frontend must not construct ComfyUI workflow JSON or contain model-specific inference logic.

### 3.2 FastAPI backend

Responsibilities:

- Validate requests and uploaded files.
- Create and track editing jobs.
- Maintain workflow state.
- Coordinate segmentation, image processing, and generation adapters.
- Provide stable application APIs independent of model vendors.
- Stream progress through WebSocket.
- Store references to source images, masks, and outputs.
- Return actionable errors without exposing secrets or unsafe filesystem paths.

The backend is the application orchestration boundary. Model services must not independently change the product workflow.

### 3.3 Segmentation / vision

Two candidates require evaluation:

- Mirage: evaluate image understanding, object detection, and mask generation. Do not assume arbitrary click/point prompting until verified.
- SAM 2 or another point-promptable segmenter: evaluate as a fallback or alternative for click-driven segmentation.

The product depends on a segmentation adapter/interface, not directly on a specific model.

A segmentation response should identify the mask, image dimensions, and relevant metadata. Validate mask dimensions against the source image before approval.

Do not make Mirage the only segmentation path until it passes the project benchmark.

### 3.4 Image processing

Technology: OpenCV and Pillow.

Responsibilities:

- Validate image and mask dimensions.
- Apply mask refinement operations.
- Apply approved feathering or edge treatment.
- Crop and prepare generation inputs when required.
- Composite the generated region back into the original image.
- Preserve original pixels outside the final approved compositing mask.
- Save intermediate and final artifacts.

Mask expansion, blur, and feathering must be explicit, testable operations. Do not silently alter the approved mask.

### 3.5 ComfyUI generation service

Responsibilities:

- Run the selected inpainting workflow.
- Accept inputs through a backend-managed adapter.
- Return job identifiers, progress, and generated output references.

The exact model and workflow are not yet approved. Benchmark candidates against the local 6 GB VRAM constraint before choosing a default.

The backend owns workflow submission and result handling. The frontend must never call ComfyUI directly.

### 3.6 Storage

Use local persistent directories mounted into relevant containers. Keep source uploads, masks, intermediate artifacts, generated candidates, and accepted outputs logically separate.

Suggested layout:

```text
data/
  uploads/
  masks/
  jobs/
  outputs/
  accepted/
```

Do not store large image binaries in an application database for MVP v1. A database is not required until a concrete need is established.

## 4. Core workflow state

Proposed lifecycle:

```text
UPLOADED
  -> SELECTION_REQUESTED
  -> MASK_CANDIDATE
  -> MASK_APPROVED
  -> GENERATION_QUEUED
  -> GENERATING
  -> RESULT_READY
  -> ACCEPTED

Alternative transitions:
MASK_CANDIDATE -> MASK_REJECTED
RESULT_READY -> REJECTED
RESULT_READY -> GENERATION_QUEUED (retry)
Any active state -> FAILED
```

Define legal transitions and test them. A rejected mask must not be used for generation. A failed retry must not overwrite a previously accepted output.

## 5. Coordinate and image contract

This is a critical correctness boundary.

- Original uploaded image dimensions are authoritative.
- The frontend may display a scaled image, but sends coordinates in the original-image coordinate system.
- The request includes enough image/job identity to ensure the point belongs to the correct source image.
- The backend rejects out-of-bounds coordinates and mismatched dimensions.
- Tests cover scaling, aspect-ratio preservation, letterboxing/padding if used, and resized browser layouts.

Define one coordinate convention (for example, origin at top-left, x increases rightward, y increases downward). Do not mix CSS pixels, canvas pixels, and original-image pixels.

## 6. Mask and compositing contract

- A candidate mask is not an approved mask.
- Generation is allowed only after explicit mask approval.
- Validate mask dimensions and format.
- The final compositing mask must be explicit and reproducible.
- Pixels outside the final compositing mask are copied from the immutable original image.
- Test pixel equality outside the compositing mask.
- Keep the original upload unchanged.

Feathering can affect pixels around the boundary. Define whether the feathered alpha is itself the final compositing mask; tests must use that exact definition.

## 7. API boundary

Exact routes and schemas will be defined in an API contract before implementation. At minimum, the application needs operations for:

- Upload an image.
- Request segmentation using an image and point.
- Retrieve/preview a candidate mask.
- Approve or reject a mask.
- Start an inpainting job with an approved mask and prompt.
- Subscribe to job progress.
- Retrieve a generated result.
- Accept, reject, or retry a result.

Do not invent additional endpoints or payload fields independently. Update the API contract and obtain approval before changing the interface.

## 8. Local deployment

Docker Compose is the MVP deployment mechanism.

- Isolate application services and model dependencies where useful.
- Use persistent volumes for user data and model caches.
- Use environment variables for configuration.
- Do not bake secrets or machine-specific absolute paths into images.
- Document GPU passthrough requirements and startup/diagnostic procedures.
- Verify container networking; use service names rather than hard-coded container IPs.
- Do not introduce Kubernetes, Redis, or cloud infrastructure without an approved requirement.

## 9. Hardware and model constraints

Initial target hardware: NVIDIA laptop GPU with 6 GB VRAM and 16 GB system RAM.

Required feasibility measurements:

- Model download size and disk footprint.
- Peak VRAM and system RAM.
- Cold-start/model-load time.
- Inference latency.
- Output quality on a small fixed test set.
- Stability across repeated runs.

Prefer sequential model execution and low-VRAM/crop-based workflows when measurements support them. CUDA availability alone does not prove a model will fit.

## 10. Testing strategy

Minimum test areas:

1. Image upload validation and file safety.
2. Coordinate conversion and bounds checking.
3. Segmentation adapter request/response validation.
4. Mask dimension and format validation.
5. Mask approval/state-transition rules.
6. Compositing and pixel preservation outside the final mask.
7. Generation adapter error handling.
8. Job progress and failure handling.
9. End-to-end local workflow using a mock generation provider.
10. Docker Compose startup and service health.

Use mocks for deterministic application tests. Keep a separate small hardware benchmark for real model execution.

## 11. Architecture decision rules

- This document is the provisional architecture baseline.
- ChatGPT is the architecture owner/orchestrator for this project; the user makes final product decisions.
- Coding agents may recommend changes but may not silently implement architecture changes.
- A change proposal states: problem, proposed change, affected components/files, trade-offs, migration impact, and tests.
- Obtain explicit approval before implementing a material architecture change.
- Record approved changes here or in a decision record.
