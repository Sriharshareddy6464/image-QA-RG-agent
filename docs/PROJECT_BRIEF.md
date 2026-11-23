# IQARG — Project Brief

> Working title: Image QA Regeneration  
> Repository: `iqarg-agent`  
> Status: MVP v1 — specification and local feasibility stage

## 1. Product objective

Build a locally hosted image-editing application that lets a user select a specific object or region in an image, inspect and refine its mask, describe the desired change, and regenerate only the selected area.

The user must be able to review the mask before generation and compare the result with the original image.

## 2. Core user workflow

1. Upload an image.
2. Display it on an interactive canvas.
3. Click a point to identify the intended object or region.
4. Generate a candidate segmentation mask.
5. Preview and refine the mask; allow approval or rejection.
6. Enter an editing prompt.
7. Submit the image, approved mask, and prompt for inpainting.
8. Display generation progress.
9. Review the result against the original.
10. Accept the result or retry with a revised mask or prompt.

## 3. MVP v1 scope

### Must have

- Local application startup through Docker Compose.
- Image upload and preview.
- Canvas interaction with correct mapping between displayed coordinates and original-image coordinates.
- Point-based object/region selection.
- Mask preview and basic refinement or correction.
- Prompt input for the requested edit.
- Inpainting through a local ComfyUI service.
- Progress updates from backend to frontend.
- Before/after comparison.
- Accept, reject, and retry workflow.
- Persistent local storage for uploaded images, masks, and generated outputs.
- Clear errors when a model or service is unavailable.

### Quality requirements

- The original uploaded image remains immutable.
- The user approves the mask before generation.
- Pixels outside the final approved compositing mask remain unchanged in the delivered result.
- Coordinate transformations are tested for scaling, cropping, and canvas resizing.
- Failed generation must not overwrite the original or last accepted result.
- The UI distinguishes candidate mask, approved mask, and generated result.

### Explicitly out of scope for MVP v1

- AMD Cloud GPU deployment. Preserve the available AMD cloud credits for a later phase.
- Kubernetes or multi-node orchestration.
- User accounts, teams, billing, and collaboration.
- Public hosting or multi-tenant production deployment.
- Model training or fine-tuning.
- Distributed job queues and autoscaling.
- Multiple cloud inference providers.
- Batch editing of large image collections.
- A general-purpose image-generation platform.

## 4. Initial technical direction

The following is provisional and subject to local feasibility tests:

| Area | Initial choice |
|---|---|
| Frontend | React, Vite, TypeScript |
| Canvas | Konva.js or equivalent |
| API backend | Python, FastAPI |
| Image operations | OpenCV and Pillow |
| Generation | ComfyUI through its API |
| Progress | WebSocket |
| Local orchestration | Docker Compose |
| Persistence | Local directories mounted as Docker volumes |
| Segmentation | Benchmark Mirage and a point-promptable fallback such as SAM 2 |
| Version control | Git |

The frontend communicates with the backend through application APIs. It must not depend directly on ComfyUI workflow details.

## 5. Hardware constraints

Initial development machine:

- Windows 11
- NVIDIA laptop GPU with 6 GB VRAM
- 16 GB system RAM
- Docker Engine and Docker Compose
- WSL2

Model feasibility is a first-class requirement. Do not assume a high-memory diffusion model will run reliably. Benchmark model loading, inference, latency, and peak memory before selecting the default workflow.

Prefer sequential model execution, low-VRAM configurations, and crop-based inpainting where justified. Do not keep multiple large models resident in GPU memory without measured evidence.

## 6. Key risks to validate

1. Segmentation quality: can the model identify the intended object from a user click?
2. Mask control: can the user inspect and correct masks before generation?
3. Generation feasibility: can inpainting run within 6 GB VRAM and 16 GB RAM?
4. Pixel preservation: are pixels outside the approved compositing mask unchanged?
5. Coordinate correctness: does a displayed canvas click map to the correct original-image pixel?
6. Runtime reliability: can services start, communicate, and recover through Docker Compose?

## 7. Definition of MVP success

A user can upload a supported image, click an object, inspect and approve its mask, submit an edit prompt, receive an inpainted result, and compare it with the original — all locally — without corrupting the source image.

Automated tests cover coordinate mapping, mask handling, compositing/pixel preservation, API validation, and the main workflow.

## 8. Decision log

Record approved architectural decisions in `ARCHITECTURE.md` or a separate decision record. Exploratory ideas, model claims, and untested assumptions are not approved decisions.

| Decision | Current status |
|---|---|
| MVP runs locally using Docker Compose | Approved direction |
| AMD Cloud GPU | Deferred |
| Mirage as the only segmentation model | Not approved; benchmark first |
| SAM 2 as mandatory segmentation model | Not approved; benchmark first |
| Exact inpainting model/workflow | Pending hardware benchmark |
| Final canvas library | Provisional; confirm during frontend spike |
