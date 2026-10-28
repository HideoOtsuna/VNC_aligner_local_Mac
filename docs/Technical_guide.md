# VNC Aligner: technical guide

For a first alignment, follow the [step-by-step README](../README.md). This guide covers installation details, settings, batch processing, and troubleshooting.

VNC alignment pipeline by Hideo Otsuna. It preprocesses ventral nerve cord images in Fiji, registers them to a reference template with CMTK, resamples the image channels, calculates an alignment score, and generates color-depth projections.

This repository includes a Fiji application with the custom plugins, preprocessing and output macros, projection masks, macOS CMTK executables, and the registration templates needed by the default unisex workflow. **Extract the Fiji archive before running; the script locates its dependencies relative to the repository folder.**

## Download

`Fiji_VNC_aligner.zip` is approximately 509 MiB. The Fiji archive and both NRRD files in `Template/` are stored using Git Large File Storage (Git LFS).

- **GitHub Desktop:** clone [this repository](https://github.com/HideoOtsuna/VNC_aligner_local_Mac) with GitHub Desktop, which includes [Git LFS support](https://docs.github.com/en/desktop/configuring-and-customizing-github-desktop/about-git-large-file-storage-and-github-desktop).
- **Terminal:** install [Git LFS](https://git-lfs.com/) first, then run:

```bash
git lfs install
git clone https://github.com/HideoOtsuna/VNC_aligner_local_Mac.git
cd VNC_aligner_local_Mac
git lfs pull
```

GitHub's **Code > Download ZIP** may contain LFS pointers instead of the Fiji archive and NRRD templates; this depends on the repository's [archive settings](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/managing-repository-settings/managing-git-lfs-objects-in-archives-of-your-repository). Use a clone with Git LFS to retrieve the full files. A small text file beginning with `version https://git-lfs.github.com/spec/v1` is a pointer, not an archive or template volume.

Extract `Fiji_VNC_aligner.zip` into the repository folder. On macOS, double-click it in Finder, or run this command from the repository folder:

```bash
ditto -x -k Fiji_VNC_aligner.zip .
```

## Repository contents

```text
VNC_aligner_local_Mac/
  20x40xVNC_Align_CMTK_localMac_LSF.sh
  Fiji_VNC_aligner.zip
  Fiji_VNC_aligner.app/                 # created by extracting the ZIP
  Template/
    JRC2018_VNC_UNISEX_1micron.nrrd
    JRC2018_VNC_UNISEX_447_G15.nrrd
    VNC_Lateral_M.tif
    VNC_Lateral_F.tif
  Copy_to_other_Fiji/
    VNC_preImageProcessing_Pipeline_05_22_2026.ijm
    nrrd_compression.ijm
    Score_Generator_Cluster.ijm
    Color_Depth_MIP_batch_0404_2019_For_Pipeline_packbits.ijm
    Fiji_plugins/
    Fiji_luts/
    Template_MIP/
    For_Score/
    cmtk/
    OLD/VNC_preImageProcessing_Pipeline_07_29_2022.ijm
  VNC_Aligner_User_Manual.md
  VNC_Aligner_User_Manual.pdf
  LICENSE
```

`Template_MIP/` contains projection masks. `For_Score/` contains scoring masks; the current scoring macro sets `UseMask=false`. Input images and alignment results are not included.

## Requirements

- macOS compatible with the bundled Intel/x86_64 Fiji, Java, and CMTK binaries. Check compatibility on the computer used for alignment.
- The extracted Fiji application. It includes Java 8 (`1.8.0_202`), the custom plugins and LUT, Bio-Formats, NRRD reading/writing, CLAHE, and Anisotropic Diffusion 2D.
- The four CMTK 3.3.1 executables in `Copy_to_other_Fiji/cmtk/`: `registration`, `warp`, `reformatx`, and `make_initial_affine`.
- The included `Template/` folder, containing these files for the default unisex workflow:

```text
Template/
  JRC2018_VNC_UNISEX_1micron.nrrd
  JRC2018_VNC_UNISEX_447_G15.nrrd
  VNC_Lateral_M.tif
  VNC_Lateral_F.tif
```

These four files total approximately 219 MiB. The 1-micron template is used for initial registration, and the 447_G15 template is used for resampling and scoring. The TIFFs provide lateral reference images during preprocessing. Other templates and deformation fields from the original collection are not needed for this default workflow and are not included. The separate `Copy_to_other_Fiji/Template_MIP/` folder contains projection masks.

The bundled Fiji already contains the custom plugins. If using another Fiji installation, install the add-ons from `Copy_to_other_Fiji/Fiji_plugins/` and the LUT from `Copy_to_other_Fiji/Fiji_luts/`. Preserve the plugin subfolders and replace existing versions of the same plugins rather than installing duplicate copies. Restart Fiji afterward.

## Configure the paths

The local configuration in `20x40xVNC_Align_CMTK_localMac_LSF.sh` finds the repository folder from the script's location. It uses the extracted Fiji app, `Copy_to_other_Fiji/`, and the included `Template/` folder without requiring a machine-specific path.

Keep `testmode=1`: this selects local execution and runs the complete alignment. The relevant assignments in the local `if [[ $testmode == "1" ]]; then` block are shown below. Edit them only if using a different Fiji installation or template folder, replacing the existing assignments in place.

```bash
REPO_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
MACRO_DIR="${REPO_DIR}/Copy_to_other_Fiji/"
FIJI_APP="${REPO_DIR}/Fiji_VNC_aligner.app"
FIJI="${FIJI_APP}/Contents/MacOS/ImageJ-macosx"
JAVA_HOME="${FIJI_APP}/java/macosx/adoptopenjdk-8.jdk/jre/Contents/Home"
CMTK="${MACRO_DIR}cmtk"

TempDir="${REPO_DIR}/Template"
MIPTempDir="${MACRO_DIR}Template_MIP"
PREPROCIMG="${MACRO_DIR}VNC_preImageProcessing_Pipeline_05_22_2026.ijm"
NRRDCOMP="${MACRO_DIR}nrrd_compression.ijm"
```

`MACRO_DIR` must end with `/`; `FIJI` must point to the executable inside the app. Use absolute paths without spaces or commas for the repository, templates, input images, and output folders, because the script and macros do not consistently support those characters.

Keep `TEMPNUMBER=1` and `INPUT1_GENDER="u"` for the demonstrated unisex configuration.

## Check the installation

The shell script specifies the Fiji application with `FIJI_APP` and automatically exports its bundled `JAVA_HOME` and `PATH` for each run. No Java setup commands are needed in Terminal. If using another Fiji application, edit `FIJI_APP` and the corresponding `JAVA_HOME` path inside the script.

From the repository folder, these optional checks confirm the shell syntax and CMTK version:

```bash
cd /absolute/path/VNC_aligner_local_Mac
bash -n 20x40xVNC_Align_CMTK_localMac_LSF.sh
./Copy_to_other_Fiji/cmtk/registration --version
```

The bundled Java version is `1.8.0_202`; CMTK reports `3.3.1`. `bash -n` should exit successfully without output. The script checks that Fiji and its bundled Java exist before processing. Confirm that all four files listed above are present in `Template/` and that the NRRD files have been downloaded through Git LFS.

## Run an alignment

From the repository folder, after extracting Fiji:

```bash
bash 20x40xVNC_Align_CMTK_localMac_LSF.sh \
    /absolute/path/sample.lsm \
    8 \
    /absolute/path/output_run1
```

The arguments are **the full file path to one confocal stack, thread count, and output folder**. The input path must start with `/`; relative input paths are rejected. Use at least 2 threads; 8 is the example setting. Despite `LSF` in the filename, local execution does not require a cluster or `bsub`.

To process every lowercase `.lsm` file directly in a folder, sequentially:

```bash
out="/absolute/path/output_batch_$(date +%Y%m%d_%H%M%S)"
for file in /absolute/path/input_folder/*.lsm; do
    [ -f "$file" ] || continue
    bash 20x40xVNC_Align_CMTK_localMac_LSF.sh "$file" 8 "$out" || break
done
```

Use a fresh output folder for each rerun. The script skips some stages when output paths already exist, so reusing an interrupted run can reuse incomplete results. Samples in a batch need distinct filename stems.

## Current processing settings

- The workflow supports **2–4 input channels**: one anatomical reference and one to three signal channels. Preprocessing, CMTK resampling, and projection generation handle up to four channels.
- The active preprocessing macro is `VNC_preImageProcessing_Pipeline_05_22_2026.ijm`. The older 2022 macro remains under `Copy_to_other_Fiji/OLD/`.
- The local runner sets `nc82decision="Signal_amount"`. The macro measures the mean intensity of each channel's **sum-slice projection** and selects the channel with the largest value as the reference. For equally sized channels, this ranks their total voxel intensity. Equal values select the first matching channel. The selected reference becomes output `_01`; for a two-channel input, the other channel becomes `_02`, regardless of input order.
- `preproc.log` records the reference-selection mode, each channel's measured signal amount, and the selected channel (`ch1 is reference`, `ch2 is reference`, etc.). Verify that the detected channel contains the anatomical reference. To force a particular channel, set the local runner's `nc82decision` to `ch1`, `ch2`, `ch3`, or `ch4` for a channel present in the input. The `sr`/`ssr` argument no longer overrides this setting.
- `VNC_DV_FLIP=off` is hard-coded in the shell script. An environment variable cannot override it. Confirm `DV flip mode: off` in the preprocessing log and inspect the resulting orientation.
- The 2026 macro forces shape analysis off. Changing the shell's shape-analysis setting alone does not override this macro decision.
- The default target is `JRC2018_VNC_UNISEX_447_G15.nrrd`, with initial registration to `JRC2018_VNC_UNISEX_1micron.nrrd`.

## Results and validation

For a two-channel input named `sample.lsm`, the main outputs are:

```text
output_run1/
  sample/
    sample_01.nrrd
    sample_02.nrrd
    JRC2018_VNC_UNISEX_447_G15_Score.property
    debug/preproc.log
  MIP/
    sample_01.tif
    sample_02.tif
```

For three- or four-channel inputs, additional aligned volumes end in `_03.nrrd` and `_04.nrrd`, with corresponding projections in `MIP/`. The selected reference is always `_01`; the remaining input channels retain their relative input order in `_02` through `_04`.

The sample folder also contains preprocessing intermediates, CMTK transforms, and any generated error properties. The script does not create a `FinalOutputs` folder.

Check that both aligned volumes open, both projections exist, and the score file contains a number. Compare the reference anatomy and slice direction against the template. A successful shell exit or high score alone does not establish a valid alignment.

The repackaged Fiji archive has had its dependencies checked but has not been validated with a new full alignment. Other acquisition types and template modes require separate validation. The script's female/male initialization branches need repair, and template option 3 contains a filename typo; changing only the template or gender settings is insufficient.

## Further documentation

The [Markdown manual](../VNC_Aligner_User_Manual.md) and [PDF manual](../VNC_Aligner_User_Manual.pdf) contain additional workflow and troubleshooting details. They describe the earlier local package and retain its old paths, external-template instructions, and some filenames that are absent here. Use this guide for the current repository layout, bundled Fiji, included templates, Java setup, and path configuration.

See [LICENSE](../LICENSE) for the repository license.
