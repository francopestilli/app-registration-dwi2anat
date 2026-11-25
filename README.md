# DWI-to-Anatomical MRI Registration

<!-- brainlife-app:begin (generated from the App manifest; edit the manifest, not this block) -->

> **Registers DWI to an anatomical reference (T1w, T2w or FA) with ANTs, using the b0 volume and translation, rigid, affine or SyN stages, applies the transform to the full DWI with MRtrix3 and produces QC figures.**

Extracts the first b=0 (or lowest-b) volume, registers it to the chosen anatomical image with antsRegistration up to the selected transformation stage and pyramid depth, then applies the affine and warp to the 4-D DWI (with b-vector handling) and writes before/after QC figures. Provide one fixed image: T1w, T2w or a tensor dataset (FA is used). Optionally checks white-matter overlap on FA before and after registration (qc_wm).

| | |
|---|---|
| 📥 **Inputs** | `neuro/dwi` · `neuro/anat/t1w` (optional) · `neuro/anat/t2w` (optional) · `neuro/tensor` (optional) |
| 📤 **Outputs** | `neuro/transform/nifti` · `neuro/dwi` · `generic/images` |
| 🐳 **Container** | `gamorosino/tract_align:v1.1` |
| ⚙️ **Resources** | 1 CPUs · 2 h walltime |
| 🔗 **Run it** | [doi.org/10.25663/brainlife.app.857](https://doi.org/10.25663/brainlife.app.857) |

#### 📥 Inputs

| Input | Datatype | Required | Description |
|---|---|---|---|
| `dwi` | `neuro/dwi` | yes | 4-D diffusion-weighted image with bvals and bvecs (moving image). |
| `t1_fixed` | `neuro/anat/t1w` | no | T1-weighted image used as the fixed reference. |
| `t2_fixed` | `neuro/anat/t2w` | no | T2-weighted image used as the fixed reference if no T1w is given. |
| `tensor_fixed` | `neuro/tensor` | no | Tensor dataset whose FA map is used as the fixed reference if no anatomical image is given. |

#### 📤 Outputs

| Output | Datatype | Description |
|---|---|---|
| `transformations` | `neuro/transform/nifti` | ANTs affine matrix and warp fields mapping the DWI to the anatomical reference. |
| `transformed` | `neuro/dwi` | DWI resampled into the anatomical reference space, with bvals and bvecs. |
| `figures` | `generic/images` | Before/after registration QC figures (orthogonal views and slices). |

<details>
<summary><b>⚙️ Parameters</b> (3)</summary>

| Parameter | Type | Default | Description |
|---|---|---|---|
| `settings` | enum | `4` | The `settings` parameter controls the depth of the multiresolution (pyramidal) registration strategy used by ANTs. Higher settings correspond to more levels in the registration pyramid, starting from coarse alignment and progressively refining toward finer spatial correspondence. Options: `1`, `2`, `3`, `4`. |
| `transformation` | enum | `rigid` | Highest registration stage to run: translation, rigid, affine or nonlinear (SyN); each stage includes the previous ones. Options: `translation`, `rigid`, `affine`, `nonlinear`. |
| `qc_wm` | boolean | `false` | Fits a diffusion tensor model on both the pre- and post-registration DWI, thresholds FA ≥ 0.35 to produce a white matter mask, and overlays it in blue on all QC images so you can verify that the WM geometry aligns with the reference anatomy after registration. |

</details>

<!-- brainlife-app:end -->

A reproducible and automated pipeline to align diffusion-weighted imaging (DWI) data to anatomical MRI references (e.g., T1-weighted) using configurable transformation stages. The workflow includes b=0 extraction, multi-stage registration (translation, rigid, affine, nonlinear), and full-volume transformation, ensuring spatial alignment suitable for tractography and cross-modal analyses.

---

## Author

**Gabriele Amorosino**
(email: [gabriele.amorosino@utexas.edu](mailto:gabriele.amorosino@utexas.edu))

---

## Description

This pipeline extracts the b=0 volume from a 4D DWI dataset, registers it to a selected anatomical image using ANTs, and applies the computed transformation to the full DWI image using MRtrix3. The workflow is configured via a `config.json` file and supports execution in containerized environments (Singularity), ensuring portability and reproducibility.

---

## Requirements

- [Singularity](https://sylabs.io/guides/latest/user-guide/)

---

## Usage

### Running on Brainlife.io

#### On Brainlife.io via Web UI

1. Navigate to the [Brainlife.io](https://brainlife.io) platform and locate the `app-registration-dwi2anat` app.
2. Click the **Execute** tab.
3. Upload required inputs:
   - 4D DWI NIfTI file (`.nii.gz`)
   - Associated `.bval` and `.bvec` files
   - An anatomical image (e.g., T1w, T2w, or FA)
   - Optionally, a `config.json` to override default parameters
4. Submit the job. Output will include registered DWI images and associated transformation matrices and fields.

#### On Brainlife.io via CLI

1. Install the Brainlife CLI: https://brainlife.io/docs/cli/
2. Log in:
   ```bash
   bl login
   ```
3. Run the app:
   ```bash
   bl app run --id <app_id> --project <project_id> \
     --input dwi:<dwi_id> \
     --input bvals:<bval_id> \
     --input bvecs:<bvec_id> \
     --input t1:<t1_id>
   ```

Replace IDs with the appropriate dataset and project references.

---

### Running Locally

#### Option 1: Using a Configuration File

1. Clone the repository:
   ```bash
   git clone https://github.com/gamorosino/app-registration-dwi2anat.git
   cd app-registration-dwi2anat/main
   ```

2. Create a `config.json`:
   ```json
   {
       "dwi": "sub-01_dwi.nii.gz",
       "bvals": "sub-01.bval",
       "bvecs": "sub-01.bvec",
       "t1": "sub-01_T1w.nii.gz",
       "transformation": "nonlinear",
       "settings": "2"
   }
   ```

3. Run the pipeline:
   ```bash
   bash ./main
   ```

---

## Outputs

- `b0/` — Extracted b=0 volume
- `ANTs_outputs/` — ANTs registration outputs: affine matrix, warp fields
- `transformations/` — Final transformation files (formatted for Brainlife)
- `transformed/` — Fully aligned DWI image



---

## Citation

If you use this repository in your work, please cite the tools and frameworks leveraged:

- Avants et al., *Advanced Normalization Tools (ANTs)*, Insight Journal, 2009.  
- Tournier et al., *MRtrix3: A fast, flexible and open software framework for medical image processing and visualisation*, NeuroImage, 2019. 

---

## License

This project is released under the [MIT License](LICENSE).
